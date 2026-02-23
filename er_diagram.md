# Database Schema

```mermaid
---
config:
    layout: elk
---
erDiagram

oauth_token {
    int oauth_id PK
    varchar(40) user_id FK 
    varchar(50) oauth_provider
    varchar(255) oauth_provider_id
    datetime created_at
    datetime updated_at
}

user_setting {
    varchar(40) user_setting_id PK
    varchar(40) user_id FK
    varchar(10) theme
    varchar(3) language
    datetime created_at
    datetime updated_at
}

user {
    varchar(40) user_id PK
    varchar(60) username
    varchar(255) email
    datetime created_at
    datetime updated_at
}

tag {
    int tag_id PK
    id note_id FK
    varchar(20) tag
}

location {
    varchar(40) location_id PK
    varchar(40) reminder_id FK
    float latitude
    float longitude
    float radius
}

reminder {
    varchar(40) reminder_id PK
    varchar(40) user_id FK
    varchar(40) note_id
    varchar(40) location_id
    datetime created_at
    datetime updated_at
}

note {
    varchar(40) note_id PK
    varchar(40) reminder_id FK
    varchar(255) title
    text description
    datetime created_at
    datetime updated_at
}

notification {
    varchar(40) notification_id PK
    varchar(40) user_id FK
    int reminder_id FK
    datetime last_triggered_at
    varchar(10) is_active
}

reminder_recurrence {
    int recurrence_id PK
    int reminder_id FK
    varchar(10) recurrence_type
}

user ||--o{ reminder : has
user ||--o{ oauth_token : has
user ||--|| user_setting : has
reminder ||--o{ note : has
note ||--o{ tag : has
reminder ||--|| location : has
reminder ||--|| notification : has
reminder ||--o| reminder_recurrence : might_have
```
