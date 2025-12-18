# LMS / LXP – Course Listing & Visibility Flow

This document describes how the React-based LMS/LXP UI, AEM LMS Service, and
External Configuration (via App Builder + External DB) work together to fetch
courses and determine visibility, actions, and access deadlines.

---

## Sequence Diagram – Course Fetching & Decision Making

```mermaid
sequenceDiagram
    participant Learner
    participant ReactLXP as React LXP (UI)
    participant Auth as Auth / Identity Service
    participant AEM as AEM LMS Service
    participant Course as Course & Enrollment Data (AEM)
    participant AppBuilder as App Builder Service
    participant ExtDB as External Config DB
    participant Rules as Business Rules Engine

    Learner->>ReactLXP: Open Course Listing Page

    ReactLXP->>Auth: Fetch Logged-in Learner Context
    Auth-->>ReactLXP: Learner ID, Roles, Locale

    ReactLXP->>AEM: GET /courses?learnerId
    AEM->>Course: Fetch Courses, Instances, Enrollments, Progress
    Course-->>AEM: Course, Enrollment, Progress Data

    AEM->>AppBuilder: Fetch External Configurations
    AppBuilder->>ExtDB: Query Config and Overrides
    ExtDB-->>AppBuilder: Config Data
    AppBuilder-->>AEM: Normalized Configuration Response

    Note right of ExtDB:
        Catalog visibility flags
        Role and region eligibility
        Re-enroll enablement
        Re-enroll time limits
        Access deadline overrides
        Feature flags

    AEM->>Rules: Evaluate Course Visibility and Actions
    Note right of Rules:
        Inputs include learner context
        Course metadata and enrollment
        Progress and external configs
        Outputs include visibility and actions

    Rules-->>AEM: Visible Courses and Allowed Actions

    AEM->>Rules: Calculate Access Deadline
    Note right of Rules:
        Instance end date
        Enrollment date
        External override rules
        Grace period
        Re-enroll window
        Previous progress retention

    Rules-->>AEM: Access Status and Deadline

    AEM-->>ReactLXP: Course List Response
    Note right of ReactLXP:
        Course metadata
        Progress percentage
        Access deadline
        Access status
        Allowed actions like Enroll, Re-enroll, View

    ReactLXP-->>Learner: Render Courses and Actions
