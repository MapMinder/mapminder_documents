# MapMinder Design System (WIP)

A comprehensive design reference for the MapMinder location-based reminder application.

---

## 1. Design Philosophy

MapMinder follows a **"map-first, mobile-native"** design philosophy. The map is the hero of the interface - it's not just a feature, it's the entire canvas. Every UI element floats above it, appearing only when needed and disappearing when not.

### Core Principles

1. **Spatial Context** - Users think in places, not lists. The map anchors every interaction to a real-world location.
2. **Progressive Disclosure** - Show only what's needed. Details expand on demand through bottom sheets and overlays.
3. **Glanceable Status** - Color-coded markers let users understand their reminders at a glance without reading text.
4. **Touch-First** - Every interactive element is sized for fingers (minimum 44px touch targets), with generous spacing.

---

## 2. Color System

### Philosophy

The color palette uses a **dark foundation** to make the map readable at any time of day and to reduce eye strain during extended use. Accent colors are borrowed from traffic light conventions (green = go/active, yellow = caution/paused, gray = stopped/done) to create instant recognition.

### Primary Palette

| Name | Value (OKLCH) | Hex Equivalent | Usage |
|------|---------------|----------------|-------|
| **Background** | `oklch(0.12 0.01 250)` | `#1a1a2e` | App background, map container |
| **Surface** | `oklch(0.16 0.01 250)` | `#252538` | Cards, bottom sheets, modals |
| **Elevated** | `oklch(0.22 0.01 250)` | `#2a2a3e` | Raised elements, hover states |
| **Border** | `oklch(0.28 0.01 250)` | `#3a3a4e` | Dividers, card borders |

### Text Colors

| Name | Value | Usage |
|------|-------|-------|
| **Primary Text** | `oklch(0.95 0 0)` / White | Headlines, important content |
| **Secondary Text** | `oklch(0.7 0 0)` / Gray | Descriptions, metadata |
| **Muted Text** | `oklch(0.5 0 0)` / Dark Gray | Placeholders, disabled text |

### Status Colors

These colors communicate reminder states instantly:

| Status | Color | Hex | Meaning |
|--------|-------|-----|---------|
| **Active** | Green | `#4ade80` | Reminder is monitoring, will trigger when you arrive |
| **Paused** | Amber | `#fbbf24` | Reminder exists but won't trigger until resumed |
| **Completed** | Gray | `#6b7280` | Reminder has been fulfilled |
| **User Location** | Blue | `#3b82f6` | Your current position on the map |

### Semantic Colors

| Purpose | Color | Usage |
|---------|-------|-------|
| **Primary Action** | Green `#4ade80` | Main CTA buttons, positive actions |
| **Destructive** | Red `oklch(0.55 0.22 25)` | Delete buttons, warning actions |
| **Info** | Blue `#3b82f6` | Informational elements, links |

---

## 3. Typography

### Font Stack

```css
--font-sans: 'Inter', system-ui, -apple-system, sans-serif;
--font-mono: 'Geist Mono', monospace;
```

**Inter** was chosen for its:
- Excellent legibility at small sizes (important for map labels)
- Wide range of weights for hierarchy
- Optimized for digital screens
- Neutral personality that doesn't compete with map content

### Type Scale

| Element | Size | Weight | Line Height | Usage |
|---------|------|--------|-------------|-------|
| **Display** | 32px | 700 (Bold) | 1.1 | Login screen title |
| **Headline** | 24px | 600 (Semibold) | 1.2 | Section headers |
| **Title** | 18px | 600 (Semibold) | 1.3 | Card titles, sheet headers |
| **Body** | 16px | 400 (Regular) | 1.5 | Primary content |
| **Caption** | 14px | 400 (Regular) | 1.4 | Secondary info, timestamps |
| **Small** | 12px | 500 (Medium) | 1.3 | Badges, labels |

### Text Treatment

- **Headings**: Use `text-balance` to prevent orphans
- **Body text**: Use `leading-relaxed` (1.5 line height) for readability
- **All caps**: Only for small labels/badges, always with `tracking-wide`

---

## 4. Spacing System

MapMinder uses an **8px base grid**. All spacing values are multiples of 8:

| Token | Value | Usage |
|-------|-------|-------|
| `space-1` | 4px | Tight inline spacing |
| `space-2` | 8px | Icon gaps, tight padding |
| `space-3` | 12px | Small padding |
| `space-4` | 16px | Standard padding, card padding |
| `space-5` | 20px | Medium gaps |
| `space-6` | 24px | Section spacing |
| `space-8` | 32px | Large section gaps |
| `space-10` | 40px | Major section dividers |

### Safe Areas

Mobile devices have notches, home indicators, and rounded corners. We account for these:

```css
padding-top: env(safe-area-inset-top);
padding-bottom: env(safe-area-inset-bottom);
padding-left: env(safe-area-inset-left);
padding-right: env(safe-area-inset-right);
```

---

## 5. Component Library

### 5.1 Map Markers

Custom SVG markers indicate reminder locations:

```
Path: M12 2C8.13 2 5 5.13 5 9c0 5.25 7 13 7 13s7-7.75 7-13c0-3.87-3.13-7-7-7zm0 9.5c-1.38 0-2.5-1.12-2.5-2.5s1.12-2.5 2.5-2.5 2.5 1.12 2.5 2.5-1.12 2.5-2.5 2.5z
```

| State | Fill Color | Stroke | Scale |
|-------|------------|--------|-------|
| Active | `#4ade80` | None | 1x |
| Paused | `#fbbf24` | None | 1x |
| Completed | `#6b7280` | None | 0.9x |
| Selected | Current + glow | `rgba(color, 0.3)` | 1.1x |

### 5.2 Geofence Circles

Visual representation of the 200m trigger radius:

```javascript
{
  radius: 200, // meters
  fillColor: statusColor,
  fillOpacity: 0.1,
  strokeColor: statusColor,
  strokeOpacity: 0.3,
  strokeWeight: 2
}
```

### 5.3 Bottom Sheet

The primary interaction pattern for revealing content:

| Property | Value |
|----------|-------|
| Background | `rgba(37, 37, 56, 0.95)` with backdrop blur |
| Border radius | 24px (top corners only) |
| Handle | 40px wide, 4px tall, centered, gray |
| Animation | Slide up 300ms ease-out |
| Max height | 85vh |

### 5.4 Cards (Reminder Cards)

```css
background: var(--card);
border-radius: 16px;
padding: 16px;
border: 1px solid var(--border);
```

**Hover/Active state**: Subtle background lightening and border color change.

### 5.5 Buttons

| Variant | Background | Text | Border |
|---------|------------|------|--------|
| Primary | Green gradient | White | None |
| Secondary | Transparent | White | 1px border |
| Ghost | Transparent | Gray | None |
| Destructive | Red | White | None |

**Sizing**:
- Standard: 44px height, 16px horizontal padding
- Large: 52px height, 24px horizontal padding
- Icon-only: 44px x 44px

### 5.6 Status Badges

Small pills indicating reminder state:

```css
padding: 4px 8px;
border-radius: 9999px;
font-size: 12px;
font-weight: 500;
text-transform: capitalize;
```

| Status | Background | Text |
|--------|------------|------|
| Active | `rgba(74, 222, 128, 0.2)` | `#4ade80` |
| Paused | `rgba(251, 191, 36, 0.2)` | `#fbbf24` |
| Completed | `rgba(107, 114, 128, 0.2)` | `#6b7280` |

---

## 6. Google Maps Styling

The map uses a custom dark theme to blend with the app:

### Style Summary

| Feature | Color | Notes |
|---------|-------|-------|
| Land/Geometry | `#1d1d2e` | Matches app background |
| Water | `#0e1626` | Darker than land for contrast |
| Roads | `#2a2a3e` | Subtle, not distracting |
| Highways | `#3a3a4e` | Slightly brighter than local roads |
| Parks | `#1a3328` | Muted green tint |
| Labels | `#8d8d9b` | Low contrast, readable but not dominant |

### Full JSON Style

```json
[
  { "elementType": "geometry", "stylers": [{ "color": "#1d1d2e" }] },
  { "elementType": "labels.text.stroke", "stylers": [{ "color": "#1d1d2e" }] },
  { "elementType": "labels.text.fill", "stylers": [{ "color": "#8d8d9b" }] },
  { "featureType": "administrative.locality", "elementType": "labels.text.fill", "stylers": [{ "color": "#b5b5c3" }] },
  { "featureType": "poi", "elementType": "labels.text.fill", "stylers": [{ "color": "#6d6d7b" }] },
  { "featureType": "poi.park", "elementType": "geometry", "stylers": [{ "color": "#1a3328" }] },
  { "featureType": "poi.park", "elementType": "labels.text.fill", "stylers": [{ "color": "#4a8c6f" }] },
  { "featureType": "road", "elementType": "geometry", "stylers": [{ "color": "#2a2a3e" }] },
  { "featureType": "road", "elementType": "geometry.stroke", "stylers": [{ "color": "#1d1d2e" }] },
  { "featureType": "road", "elementType": "labels.text.fill", "stylers": [{ "color": "#7a7a8b" }] },
  { "featureType": "road.highway", "elementType": "geometry", "stylers": [{ "color": "#3a3a4e" }] },
  { "featureType": "road.highway", "elementType": "geometry.stroke", "stylers": [{ "color": "#2a2a3e" }] },
  { "featureType": "road.highway", "elementType": "labels.text.fill", "stylers": [{ "color": "#9a9aab" }] },
  { "featureType": "transit", "elementType": "geometry", "stylers": [{ "color": "#252538" }] },
  { "featureType": "transit.station", "elementType": "labels.text.fill", "stylers": [{ "color": "#6d6d7b" }] },
  { "featureType": "water", "elementType": "geometry", "stylers": [{ "color": "#0e1626" }] },
  { "featureType": "water", "elementType": "labels.text.fill", "stylers": [{ "color": "#4a5568" }] }
]
```

---

## 7. Iconography

All icons are **inline SVGs** for flexibility and performance. No icon library dependencies.

### Icon Sizes

| Context | Size | Stroke Width |
|---------|------|--------------|
| Navigation | 24px | 2px |
| In-button | 20px | 2px |
| Small/Badge | 16px | 2px |
| Large/Hero | 32px | 1.5px |

### Core Icons Used

| Icon | Usage |
|------|-------|
| MapPin | Reminder locations, markers |
| Plus | Add new reminder |
| List | View reminders list |
| Bell | Notifications, reminder icon |
| Check | Complete action, checkboxes |
| Pause | Pause reminder |
| Play | Resume reminder |
| Trash | Delete reminder |
| X | Close, dismiss |
| ChevronDown | Collapse, dropdown |
| Navigation | Current location button |
| LogOut | Sign out |

---

## 8. Motion & Animation

### Principles

1. **Purposeful** - Animation guides attention, never decorates
2. **Quick** - Most transitions are 200-300ms
3. **Natural** - Use ease-out for entrances, ease-in for exits

### Standard Animations

| Animation | Duration | Easing | Usage |
|-----------|----------|--------|-------|
| Fade in | 200ms | ease-out | Overlays appearing |
| Slide up | 300ms | ease-out | Bottom sheets |
| Scale | 150ms | ease-out | Button press feedback |
| Pulse | 2000ms | ease-in-out (loop) | User location indicator |

### CSS Keyframes

```css
@keyframes slideUp {
  from {
    opacity: 0;
    transform: translateY(100%);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@keyframes pulse-ring {
  0% {
    transform: scale(0.8);
    opacity: 0.8;
  }
  100% {
    transform: scale(2);
    opacity: 0;
  }
}
```

---

## 9. Layout Patterns

### Screen Structure

```
???????????????????????????????????
?  Status Bar (safe area)         ?
???????????????????????????????????
?                                 ?
?                                 ?
?         GOOGLE MAP              ?
?      (full screen)              ?
?                                 ?
?    [Floating UI Elements]       ?
?                                 ?
???????????????????????????????????
?  Bottom Sheet (expandable)      ?
?  - Handle                       ?
?  - Content                      ?
???????????????????????????????????
?  Home Indicator (safe area)     ?
???????????????????????????????????
```

### Floating Elements

Positioned absolutely over the map:

- **Top-left**: User profile/avatar
- **Top-right**: Menu, settings
- **Bottom-right**: FAB (Add reminder), Location button
- **Bottom**: Bottom sheet handle/preview

### Z-Index Scale

| Layer | Z-Index | Content |
|-------|---------|---------|
| Map | 0 | Base map layer |
| Markers | 10 | Map markers and circles |
| Floating UI | 20 | FABs, buttons over map |
| Bottom Sheet | 30 | Expandable content panel |
| Modal | 40 | Full-screen overlays |
| Toast | 50 | Notifications |

---

## 10. Accessibility

### Touch Targets

All interactive elements have a minimum touch target of **44x44px** per Apple HIG guidelines.

### Color Contrast

- Primary text on background: 15.8:1 (AAA)
- Secondary text on background: 7.2:1 (AAA)
- Status colors have sufficient contrast for their backgrounds

### Screen Reader Support

- All icons have `aria-label` descriptions
- Interactive elements use semantic HTML (`button`, `a`)
- Status changes announced via `aria-live` regions
- Form inputs have associated labels

### Reduced Motion

Users with `prefers-reduced-motion` see instant transitions instead of animations.

---

## 11. Responsive Behavior

While MapMinder is mobile-first, it adapts to larger screens:

| Breakpoint | Behavior |
|------------|----------|
| < 640px | Full mobile layout, bottom sheet |
| 640px+ | Side panel instead of bottom sheet |
| 1024px+ | Wider side panel, more map visible |

---

## 12. Brand Elements

### Logo Treatment

"MapMinder" uses:
- **Map**: Inter Bold, Primary text color
- **Minder**: Inter Bold, Green accent color

### Tagline

"Never forget a place" - displayed on login screen in muted text.

### App Icon (Concept)

A simplified map pin in green on a dark rounded square background.

---

## 13. File Reference

| File | Purpose |
|------|---------|
| `app/globals.css` | Design tokens, CSS variables, base styles |
| `app/layout.tsx` | Font loading, metadata |
| `components/login-screen.tsx` | Authentication UI |
| `components/map-view.tsx` | Map component with markers |
| `components/bottom-sheet.tsx` | Reusable sheet component |
| `components/reminder-form.tsx` | Create/edit reminder form |
| `components/reminder-card.tsx` | List item for reminders |
| `components/reminder-details.tsx` | Full detail view |
| `components/reminders-list.tsx` | Filtered reminder list |
| `components/main-screen.tsx` | Primary app screen |
| `lib/types.ts` | TypeScript interfaces |
| `lib/app-context.tsx` | Global state management |
| `lib/geofencing.ts` | Location monitoring service |

---

*Last updated: April 2026*
*Version: 1.0*
