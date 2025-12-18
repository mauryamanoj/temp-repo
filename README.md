# LMS / LXP – Course Listing & Visibility Flow

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

    Learner->>ReactLXP: Open course listing page
    ReactLXP->>Auth: Fetch logged-in learner context
    Auth-->>ReactLXP: Learner ID and roles

    ReactLXP->>AEM: GET /courses
    AEM->>Course: Fetch courses, enrollments, progress
    Course-->>AEM: Course and enrollment data

    AEM->>AppBuilder: Fetch external configurations
    AppBuilder->>ExtDB: Query config data
    ExtDB-->>AppBuilder: Configuration records
    AppBuilder-->>AEM: Normalized configs

    Note right of ExtDB: Catalog visibility flags<br/>Role and region eligibility<br/>Re-enroll enablement<br/>Access deadline overrides<br/>Feature flags

    AEM->>Rules: Evaluate visibility and actions
    Note right of Rules: Uses learner context<br/>Enrollment state<br/>Progress data<br/>External configuration

    Rules-->>AEM: Visible courses and allowed actions

    AEM->>Rules: Calculate access deadline
    Note right of Rules: Instance end date<br/>Enrollment date<br/>Grace period<br/>Re-enroll window

    Rules-->>AEM: Access status and deadline

    AEM-->>ReactLXP: Course list with actions
    ReactLXP-->>Learner: Render courses and CTAs
