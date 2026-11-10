# Apple Developer account: what it unlocks

A paid developer membership adds capabilities a free Apple ID does not have. This page says what each one is, which stage uses it, and where to read about it.

## How these fit with Unity

Unity writes the AR scene in C#. Most Apple services below are **Swift** APIs with no built-in Unity wrapper, so you reach them one of two ways:

- **Native plugin:** a small Swift or Objective-C file inside the Unity project that C# calls into. Read [Building plug-ins for iOS](https://docs.unity3d.com/Manual/PluginsForIOS.html). You first do this in Stage 8.
- **Separate Xcode target:** widgets, Live Activities and App Clips are their own small Swift programs added to the Xcode project Unity generates.

That means this roadmap teaches you a little Swift along the way. That is useful: it is the same skill you need to call any iOS feature Unity does not expose.

A simpler alternative for syncing data is your own server (Stage 11). Where a quest says "iCloud sync", either approach meets the goal.

## The capabilities

| Capability | What it gives you | First used | Docs |
|---|---|---|---|
| **One-year signing** | Builds stay installed for a year, not 7 days | Stage 1 | [setup.md](setup.md) |
| **TestFlight** | Install builds over the air; share with up to 10,000 testers | Stage 5, 9 | [TestFlight](https://developer.apple.com/testflight/) |
| **iCloud / CloudKit** | A free database tied to the user's iCloud account, synced across their devices | Stage 3 (intro), 5, 10 | [CloudKit](https://developer.apple.com/documentation/cloudkit) |
| **Push notifications** | Remote notifications. (Local, on-device reminders need no entitlement and are the place to start) | Stage 5 | [User Notifications](https://developer.apple.com/documentation/usernotifications) |
| **Siri and App Intents** | Expose app actions to Siri, Shortcuts and Spotlight | Stage 5, 12 | [App Intents](https://developer.apple.com/documentation/appintents) |
| **Widgets** | Glanceable data on the home and lock screen | Stage 11 | [WidgetKit](https://developer.apple.com/documentation/widgetkit) |
| **Live Activities** | A live-updating lock-screen and Dynamic Island view, good for timers | Stage 3 (Focus pet), 11 | [ActivityKit](https://developer.apple.com/documentation/activitykit) |
| **WeatherKit** | Weather data; 500,000 calls a month included with membership | Stage 3 (Miniature weather) | [WeatherKit](https://developer.apple.com/weatherkit/) |
| **App Clips** | A tiny slice of your app that launches from a code or NFC tag with no install | Stage 4 (Instant card) | [App Clips](https://developer.apple.com/app-clips/) |

## Things that are free but worth knowing

These need no paid membership, and they matter for the roadmap:

- **ARKit, RealityKit, Vision, Core ML, Speech** — all the on-device frameworks.
- **AR Quick Look** — any `.usdz` model opens in AR straight from Safari or Files with zero code. A fast way to check a 3D model before you bring it into Unity. See [AR Quick Look](https://developer.apple.com/augmented-reality/quick-look/).
- **Location anchors** — ARKit can pin content to GPS coordinates in supported cities. See [Tracking geographic locations in AR](https://developer.apple.com/documentation/arkit/arkit_in_ios/content_anchors/tracking_geographic_locations_in_ar).

## Setting up a capability

Each capability is switched on in two places, and they must match:

1. **Xcode → target → Signing & Capabilities → + Capability.**
2. The **App ID** in your [developer account](https://developer.apple.com/account/) (Xcode's automatic signing usually does this for you).

Unity regenerates the Xcode project on every build, which wipes manual capability changes. From Stage 5 on, either build with **Append** (keeps the existing Xcode project) or learn Unity's [post-process build callback](https://docs.unity3d.com/ScriptReference/Callbacks.PostProcessBuildAttribute.html) to add capabilities automatically.

## Keep out of the repo

Signing keys (`.p8`, `.p12`), provisioning profiles and API keys are git-ignored. Do not force-add them.
