[toc]

# Geofencing — Research Roadmap

> **Status:** Research / pre-implementation · **Last updated:** 2026-09-10
> **Ticket:** [MAP-98](https://linear.app/mapminder/issue/MAP-98/geofencing-integration)
> **Related:** [`reminder_flow.md`](./reminder_flow.md), [`architecture.md`](./architecture.md)

This is a study guide, not a solution. Work through the topics in order — each
builds on the last. When you can answer a topic's questions in your own words,
move on. Resolve the four [open design decisions](#open-design-decisions) before
writing code.

---

## The mental model

**Geofencing is not your app watching the GPS.** Your app hands the operating
system a list of circles ("regions") and goes to sleep. The OS watches the
hardware. When the user crosses a boundary, the OS wakes your app — even if it
was closed — and hands it an event.

It's not you at the window watching for the mail truck (battery-draining
polling). It's a note to the doorman — *"buzz me when a truck enters the lot"* —
and you go to sleep. The doorman (the OS) does the watching.

If anything below seems to contradict this, that's a sign to dig deeper there.

---

## 0. Feasibility gate — do this first

Confirm geofencing works on a **free** Apple account before investing in the
rest. If it were gated like Sign in with Apple, the whole ticket would stall.

Expected outcome: **not blocked.**

- Location permission is a runtime request, not an entitlement — no paid account.
- Background location is driven by the `UIBackgroundModes` Info.plist key, not a
  portal-gated entitlement.
- This is unlike Sign in with Apple, which needs the
  `com.apple.developer.applesignin` entitlement (paid only).

Verify: add the Background Modes → Location capability to a throwaway build with
your free Apple ID and confirm Xcode doesn't reject it.

---

## 1. CoreLocation region monitoring

The iOS feature that does the actual geofencing.

**Questions**

- What three things define a `CLCircularRegion`? (center, radius, identifier)
- What are `didEnterRegion` / `didExitRegion`, and which object receives them?
- Does monitoring fire when the app is backgrounded or terminated? What does the
  OS do to a closed app on a boundary crossing?
- What is the maximum region count, and the practical minimum radius?
- How accurate and fast is a trigger — exact and instant, or fuzzy and delayed?

**Why it matters** — the 20-region cap limits how many active reminders can have
live geofences. The "works when terminated" behaviour is the whole reason this
feature is possible, and it dictates where the notification is fired (topic 4).

**Key terms:** `CLLocationManager`, `CLCircularRegion`, region monitoring,
`startMonitoring(for:)`, `CLLocationManagerDelegate`, significant location change.

---

## 2. Flutter ↔ native (platform channels)

Dart can't call CoreLocation directly. This is the bridge.

**Questions**

- What is a `MethodChannel`, and which direction does it go — one-shot or
  streaming?
- What is an `EventChannel`, and how does it differ? Which fits a continuous
  stream of enter/exit events?
- Where does the Swift channel code live? (`ios/Runner/AppDelegate.swift`)
- What data types can cross the boundary — custom objects, or only
  primitives/maps/lists?
- How do the Dart and Swift sides agree on the channel name, and what happens if
  they don't?

**Why it matters** — a stream of events is the textbook `EventChannel` case, but
the ticket mentions `MethodChannel`. That's [open decision 1](#open-design-decisions).

**Key terms:** `MethodChannel`, `EventChannel`, `FlutterMethodChannel`,
`setMethodCallHandler`, `FlutterStreamHandler`, "Flutter platform channels".

---

## 3. iOS location permissions

Background geofencing needs the strongest location permission, and iOS makes you
earn it.

**Questions**

- "When In Use" vs "Always" — which does background/terminated geofencing need?
- Can you request "Always" directly, or does iOS force a two-step escalation
  (When In Use first, then a later Always prompt)?
- What do the two `Info.plist` usage-description keys do?
- Does `permission_handler` (already a dependency) handle the escalation, or do
  you need CoreLocation's own APIs?

**Why it matters** — with only "When In Use", the core promise (notify me with
the app closed) silently breaks. This is the most common way geofencing apps fail.

**Key terms:** `requestAlwaysAuthorization`, `requestWhenInUseAuthorization`,
`NSLocationWhenInUseUsageDescription`,
`NSLocationAlwaysAndWhenInUseUsageDescription`, provisional always authorization.

---

## 4. Local notifications

The payoff: the buzz on the phone.

**Questions**

- How do you fire a local notification from Swift? (`UNUserNotificationCenter`)
- Is notification permission separate from location permission? When do you ask?
- When the OS wakes a terminated app for a geofence event, is the Flutter engine
  running? If not, who shows the notification — Dart or Swift?
- `flutter_local_notifications` vs firing natively in Swift — trade-offs?

**Why it matters** — if Dart isn't alive when the event fires, a Dart-based
notification never shows. That likely pushes the notification into Swift.

**Key terms:** `UNUserNotificationCenter`, `UNMutableNotificationContent`,
notification authorization request, iOS local notification background.

---

## 5. The geofence-event backend round-trip

The trigger flow: enter region → `PATCH /reminder/{id}/event` → backend replies
`{ notify: true/false }` → maybe show the notification.

**Questions**

- When the OS wakes a terminated app for a region event, how much execution time
  do you get before iOS suspends you again?
- In that window, can you realistically read the JWT from secure storage, make a
  network call, wait for the response, then fire a notification? What breaks
  (no network, slow response, time runs out)?
- The cooldown lives server-side (see `reminder_flow.md`). Given the time limit,
  is "ask backend first, then notify" safe? Or notify immediately and reconcile
  later?
- What happens if the network is unavailable at the moment of the event?

**Why it matters** — the biggest design tension in the ticket. "Verify before
notifying" is clean in theory but fragile under iOS background limits. This is
[open decision 2](#open-design-decisions): correctness vs reliability.

**Key terms:** iOS background execution time limit, `beginBackgroundTask`,
URLSession background, CLRegion event background network request.

---

## 6. Integration with the reminder lifecycle

Geofences must stay in sync with reminders.

**Questions**

- Where does a reminder get created / completed / deleted in the current code?
  (`ReminderController` / `ReminderService` / `ReminderRepository`) Where do you
  hook "register geofence" and "remove geofence"?
- Why register the geofence *after* `POST /reminder` succeeds, not before?
- On launch the OS still remembers monitored regions, but app state may be stale.
  How do you reconcile the device's regions with the server's active reminders?
- With a 20-region cap, what happens when the user has more than 20 active
  reminders? (MVP may ignore this — but note it.)

**Why it matters** — a geofence that outlives its reminder, or never gets
registered, is a silent bug. Sync is where this feature rots.

**Key terms:** `locationManager.monitoredRegions`, sync geofences on app launch,
`reminder_repository.dart`.

---

## 7. Testing

You can't walk 200 m every test run.

**Questions**

- How do you simulate a location *and movement* in the iOS Simulator and on a
  real device via Xcode?
- What is a GPX file, and how do you use one to simulate crossing a boundary?
- Can you test the terminated-state path in the Simulator, or only on a real
  device?

**Why it matters** — if you can't test it cheaply, you can't build it. Know the
test loop before coding.

**Key terms:** Xcode simulate location, GPX file geofence testing, Simulator →
Features → Location, debug location Flutter iOS.

---

## Open design decisions

Resolve these before coding. Write down the decision **and the reason**.

1. **MethodChannel vs EventChannel** for enter/exit events. (Topic 2)
2. **Notify-first vs verify-first** in terminated state — correctness vs
   reliability under background limits. (Topic 5)
3. **Notification fired in Dart or native Swift** — driven by whether Flutter is
   alive when terminated. (Topic 4)
4. **Geofence sync strategy on launch** — how to reconcile device regions with
   server state. (Topic 6)

---

## Out of scope (MVP)

- Android (iOS only)
- More than 20 active geofences
- Variable radius (fixed at 200 m)
- Real-time GPS tracking
