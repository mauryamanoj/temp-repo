# LMS / LXP – Course Listing & Visibility Flow

This document describes how the **React-based LMS/LXP UI**, **AEM LMS Service**, and **External Configuration (via App Builder + External DB)** work together to fetch courses and determine visibility, actions, and access deadlines for a logged-in learner.

---

## High-Level Architecture

- **Frontend**: React-based LMS / LXP
- **Backend**: AEM LMS Service (System of Record)
- **Configuration Layer**: App Builder + External Database
- **Decision Layer**: Business Rules Engine

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
    Course-->>AEM: Course + Enrollment + Progress Data

    AEM->>AppBuilder: Fetch External Configurations
    AppBuilder->>ExtDB: Query Config & Overrides
    ExtDB-->>AppBuilder: Visibility & Rule Config Data
    AppBuilder-->>AEM: Normalized Configuration Response

    Note right of ExtDB:
        - Catalog visibility flags
        - Role / region eligibility
        - Re-enroll enablement
        - Re-enroll time limits
        - Access deadline overrides
        - Feature flags

    AEM->>Rules: Evaluate Course Visibility & Actions
    Note right of Rules:
        Inputs:
        - Learner context
        - Course metadata
        - Enrollment & progress
        - External configs
        
        Decisions:
        - Show / hide course
        - Allowed actions
        - Enrollment state

    Rules-->>AEM: Visible Courses + Allowed Actions

    AEM->>Rules: Calculate Access Deadline
    Note right of Rules:
        - Instance end date
        - Enrollment date
        - External override (DB)
        - Grace period
        - Re-enroll window
        - Previous progress carry-forward

    Rules-->>AEM: Access Status & Deadline

    AEM-->>ReactLXP: Course List Response
    Note right of ReactLXP:
        - Course metadata
        - Progress %
        - Access deadline
        - Access status
        - Actions:
          Enroll | Re-enroll | Unenroll | View | Resume

    ReactLXP-->>Learner: Render Courses & Actions
