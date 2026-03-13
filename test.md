sequenceDiagram
    participant U as Student/User
    participant LXP as LXP
    participant AB as Adobe App Builder
    participant AMEI as AMEI Authentication

    rect rgb(230,245,255)
    U->>LXP: Login
    end

    rect rgb(255,240,220)
    LXP->>AMEI: Authenticate User
    AMEI-->>LXP: Authentication Success
    end

    rect rgb(235,255,235)
    LXP-->>U: Redirect to Landing Page (Light Session)
    Note over LXP,U: UI shell loads immediately
    end

    rect rgb(240,240,255)
    LXP->>AB: Async Request → Fetch Learner Token
    AB->>AMEI: Get Learner Token
    end

    alt User Exists
        AMEI-->>AB: Learner Token
    else User Not Found
        AMEI-->>AB: User Not Found
        AB->>AMEI: Create User
        AMEI-->>AB: User Created
        AB->>AMEI: Fetch Learner Token
        AMEI-->>AB: Learner Token
    end

    rect rgb(230,255,245)
    AB-->>LXP: Return Learner Token
    LXP-->>U: Load Courses
    end

    rect rgb(255,235,235)
    AB->>AMEI: Async PATCH User Metadata
    end
