# Doodle — Project Brief for Claude Code

## What this app is

Doodle is a drawing app for a 4-year-old child. It is being built because
existing kids' drawing apps on the App Store are riddled with ads, fake
buttons that mimic app UI, and in-app purchases that interrupt the
experience. This app will be published as **free, zero ads, zero in-app
purchases**.

The primary user is a 4-year-old. Every design decision should be filtered
through that lens.

## Core principles — never compromise these

- **No network calls. Ever.** The app must function 100% offline. No
  analytics, no crash reporting SDKs, no remote config, no CDN assets.
- **No external dependencies / SPM packages** unless explicitly approved.
  Use Apple frameworks only (SwiftUI, PencilKit, SwiftData, AVFoundation).
- **Tap targets minimum 52pt** everywhere. Small children have imprecise
  touch.
- **No modal interruptions during drawing.** Alerts or sheets should only
  appear for destructive actions (clear canvas, delete artwork).
- **Sound is optional and toggleable.** Respect the sound toggle stored in
  UserDefaults key `soundEnabled`.
- **No text the child needs to read.** Icons only in the toolbar. The child
  cannot read.

## Tech stack

- SwiftUI + UIKit interop where needed
- PencilKit (PKCanvasView) for core drawing
- SwiftData for persistence
- AVFoundation for sound effects
- iOS 17+ minimum deployment target
- iPad and iPhone, both orientations

## Architecture

- `DrawingViewModel` — ObservableObject, single source of truth for drawing
  state
- `PKCanvasView` wrapped in UIViewRepresentable
- Separate overlay layers for sparkle brush and stamps (flattened on save)
- All layers flattened to a single UIImage on save
- Stroke events logged with timestamps during drawing for timelapse replay

## Project structure

```
Doodle/
  App/DoodleApp.swift
  Views/CanvasView.swift
  Views/GalleryView.swift
  Views/ToolbarView.swift
  Views/ReplayView.swift
  ViewModels/DrawingViewModel.swift
  Models/Artwork.swift          ← SwiftData model
  Models/StrokeEvent.swift
  Models/DrawingTool.swift      ← enum
  Models/BackgroundOption.swift ← enum
  Resources/Sounds/
```

## Features — build status

Update these checkboxes as each feature is completed.

- [ ] Project foundation & architecture
- [ ] Drawing toolbar (tools, colours, brush sizes, undo, clear, save, sound toggle)
- [ ] Background canvas options
- [ ] Stamps & sparkle brush
- [ ] Save & gallery
- [ ] Timelapse replay
- [ ] Polish & App Store prep

## Drawing tools

| Tool    | Implementation                                      |
|---------|-----------------------------------------------------|
| Pen     | PKInkingTool (.pen)                                 |
| Crayon  | PKInkingTool (.crayon)                              |
| Marker  | PKInkingTool (.marker)                              |
| Eraser  | PKEraserTool                                        |
| Rainbow | Custom — cycles hue as user draws                   |
| Sparkle | Custom overlay layer — scattered stars on drag      |
| Stamp   | Tap-to-place emoji, separate overlay layer          |

## Colour palette

16 kid-friendly colours — bright, saturated, easy to distinguish. Define as
a static array in DrawingTool.swift. Suggested set:

Red, Orange, Yellow, Lime Green, Green, Teal, Light Blue, Blue, Purple,
Pink, Hot Pink, White, Light Grey, Brown, Black, Gold

## Timelapse replay — data requirements

Every stroke must be logged to `[StrokeEvent]` during drawing with:
- Array of CGPoints
- Timestamp relative to drawing session start (TimeInterval)
- Tool type at time of stroke
- Colour at time of stroke

**This must be implemented from the foundation prompt — it cannot be bolted
on later without a data migration.**

Replay compresses real timestamps so total playback is capped at 30 seconds
regardless of how long the original drawing took.

## Stamp set

⭐ 🌈 🦕 🚀 🌸 ❤️ 🐶 🍕 🦋 🌙 🎈 🐸

## Background options

| Name        | Description                                          |
|-------------|------------------------------------------------------|
| White       | Plain white                                          |
| Cream       | Warm off-white                                       |
| Black       | Plain black                                          |
| Light Blue  | Soft sky blue                                        |
| Light Pink  | Soft pink                                            |
| Light Yellow| Soft yellow                                          |
| Space       | Deep purple/blue gradient with scattered star dots   |
| Ocean       | Blue gradient, lighter at top                        |
| Grass       | Green at bottom fading to light blue sky at top      |
| Blackboard  | Dark green, chalkboard style                         |

## Sound effects

Triggered on tool use, respects `soundEnabled` UserDefaults key.

| Trigger          | Sound character         |
|------------------|------------------------|
| Pen/Crayon/Marker| Soft scratching         |
| Sparkle brush    | Small chime             |
| Stamp placement  | Pop or boing            |
| Eraser           | Soft squeak             |

Use AVFoundation with bundled audio files (free/open licence from
freesound.org) or fall back to AudioServicesPlaySystemSound.

## App Store intent

- Free
- No in-app purchases
- No ads
- Age rating: 4+
- Primary category: Education

## First launch behaviour

Show a one-time tooltip (dismissed via UserDefaults flag `hasShownGuidedAccessHint`)
suggesting parents enable iOS Guided Access to prevent the child accidentally
exiting the app:

> "Tip for parents: enable Guided Access in Settings > Accessibility to
> keep your child in this app."

Never show again after dismissal.

## Things to never add without asking

- Any form of user account or login
- Any network request of any kind
- Any third-party SDK or SPM package
- Any feature requiring the child to read text
- Any purchase or monetisation flow
- Any analytics or telemetry
