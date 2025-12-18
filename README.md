# temp-repo

sequenceDiagram
    participant Learner
    participant ReactLXP as React LXP (UI)
    participant Auth as Auth / Identity Service
    participant AEM as AEM LMS Service
    participant Course as Course & Enrollment Data (AEM)
    participant AppBuilder as App Builder Service
    participant ExtDB as External Config DB
    participant Rules as Business Rules Engine

    Learner->>ReactLXP: Open Course Listing

    ReactLXP->>Auth: Fetch Logged-in Learner Context
    Auth-->>ReactLXP: Learner ID, Roles, Locale

    ReactLXP->>AEM: GET /courses?learnerId
    AEM->>Course: Fetch Courses, Instances, Enrollments
    Course-->>AEM: Course + Enrollment + Progress Data

    AEM->>AppBuilder: Fetch External Configs
    AppBuilder->>ExtDB: Query Config & Overrides
    ExtDB-->>AppBuilder: Config Data
    AppBuilder-->>AEM: Normalized Config Response

    Note right of ExtDB:
        - Catalog visibility flags
        - Course eligibility rules
        - Custom enrollment rules
        - Re-enroll enablement
        - Access deadline overrides
        - Region / role mappings

    AEM->>Rules: Evaluate Visibility & Actions
    Note right of Rules:
        Inputs:
        - Course metadata (AEM)
        - Enrollment status
        - Learner context
        - External configs (App Builder)
        
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
        - Previous progress retention

    Rules-->>AEM: Access Status & Deadline

    AEM-->>ReactLXP: Course List Payload
    Note right of ReactLXP:
        - Course info
        - Progress %
        - Access deadline
        - Access state
        - Actions:
          Enroll | Re-enroll | Unenroll | View | Resume

    ReactLXP-->>Learner: Render Courses & Actions
