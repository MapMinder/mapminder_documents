# Reminder feature flow

# TOC
- [TOC](#toc)
    - [sequence](#sequence)
    - [Geofencing – iOS MVP Plan](#geofencing--ios-mvp-plan)
        - [Scope](#scope)
        - [Architecture](#architecture)
        - [Permissions](#permissions)
        - [Behavior](#behavior)

## sequence

```mermaid
sequenceDiagram
actor user
participant mf as app
participant gf as geofencing
participant mb as server
participant db as database

user->>mf: create reminder (title, location, radius)
mf->>mb: POST /reminders
mb->>db: insert reminder (status='active')
db->>mb: return created reminder
mb->>mf: return reminder (id + server fields)
mf->>gf: register geofence (lat, lng, radius, reminder_id)
gf->>mf: geofence registered
mf->>user: show confirmation


Note over user,gf: Later, user enters geofence

user->>gf: enters geofence
gf->>mf: geofence event (reminder_id)

mf->>mb: POST /reminders/{id}/event { type: "geofence_entered" }

mb->>db: update<br/>where status='active'<br/>and last_triggered_at IS NULL<br/>or cooldown passed

alt cooldown passed and active
    db->>mb: row updated (last_triggered_at=now)
    mb->>mf: { notify: true }
    mf->>user: show local notification
else inactive or in cooldown
    db->>mb: no rows updated
    mb->>mf: { notify: false }
end

user->>mf: mark reminder as complete
mf->>mb: PATCH /reminders/{id} { status: "completed" }
mb->>db: update status='completed', completed_at=now
db->>mb: success
mb->>mf: success
mf->>gf: remove geofence
mf->>user: show completion confirmation
```
---

## Geofencing – iOS MVP Plan
### Scope

- Platform: iOS only
- Framework: Flutter (UI)
- Native layer: Swift
- No third-party geofencing plugins

### Architecture
Flutter → MethodChannel → Native Swift → CoreLocation (Region Monitoring)

**Flutter:**
- Sends latitude, longitude, radius, identifier
- Receives enter/exit events if app is active

**Swift:**
- Registers CLCircularRegion
- Handles didEnterRegion / didExitRegion
- Triggers local notification if app is backgrounded or terminated

### Permissions
**Required:**
- Always Location permission
- Background mode: Location updates

**Info.plist:**
- NSLocationWhenInUseUsageDescription
- NSLocationAlwaysAndWhenInUseUsageDescription

### Behavior
- Works in foreground, background, and terminated state
- Max 20 regions
- Radius ≥ 100m
- Not real-time GPS tracking
