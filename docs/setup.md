# Setup

What to install and how to get a Unity scene onto your iPhone. You do this during Stage 1; Stage 0 needs only a browser and a text editor.

## Stage 0 needs

- A code editor ([VS Code](https://code.visualstudio.com/) is fine).
- A way to serve a folder over HTTPS to your phone. Camera access in Safari requires HTTPS on anything other than `localhost`. Options are in the [Stage 0 brief](../projects/00-web-image-tracking/README.md).
- A printer, or a second screen to display the target image.

## Stage 1 onward

### 1. Xcode

Install Xcode from the Mac App Store, open it once, and let it install the iOS platform. Sign in under **Xcode → Settings → Accounts** with the Apple ID that holds your developer membership.

### 2. Unity

1. Install [Unity Hub](https://unity.com/download).
2. In the Hub, install the latest **Unity 6 LTS** editor. On an Apple Silicon Mac choose the Apple Silicon build.
3. Tick the **iOS Build Support** module during install.
4. Unity Personal is free and is all this roadmap needs.

### 3. A project with AR Foundation

1. New project → **Universal 3D** template → save it inside the stage folder (for example `projects/02-tap-to-place/TapToPlace`).
2. **Window → Package Manager → Unity Registry**: install **AR Foundation** and **Apple ARKit XR Plugin**.
3. **Edit → Project Settings → XR Plug-in Management → iOS tab**: tick **ARKit**.
4. **Project Settings → Player → iOS**: set a **Camera Usage Description** (any sentence), and a unique **Bundle Identifier** such as `com.yourname.taptoplace`.

The AR Foundation manual walks through the same steps with screenshots: [project setup](https://docs.unity3d.com/Packages/com.unity.xr.arfoundation@6.0/manual/project-setup/project-setup.html).

### 4. Build to the phone

1. **File → Build Profiles → iOS → Switch Platform**, then **Build** into a folder named `iOSBuild` (already git-ignored).
2. Open the generated `.xcodeproj` in Xcode.
3. Select the **Unity-iPhone** target → **Signing & Capabilities** → tick **Automatically manage signing** and pick your developer team.
4. Plug in the iPhone, trust the Mac, enable **Developer Mode** on the phone (Settings → Privacy & Security), and press Run.

Unity's guide: [Getting started with iOS](https://docs.unity3d.com/Manual/iphone-GettingStarted.html).

AR does not run in the Unity editor's Play mode with a real camera. Expect the loop to be: change → build → run on phone. Keep builds small and test one thing at a time. The [XR Simulation](https://docs.unity3d.com/Packages/com.unity.xr.arfoundation@6.0/manual/xr-simulation/simulation-overview.html) feature lets you test plane and image logic in the editor against a fake room, which saves a lot of builds.

### 5. Git and LFS

```bash
brew install git-lfs
```

```bash
git lfs install
```

The root `.gitignore` and `.gitattributes` already cover Unity. Before your first Unity commit, check **Project Settings → Editor**: Asset Serialization should be **Force Text** and Version Control mode **Visible Meta Files** (both are the defaults in Unity 6).

Commit the `Assets/`, `Packages/` and `ProjectSettings/` folders. Never commit `Library/`.

### 6. TestFlight (from Stage 5)

1. Create the app record in [App Store Connect](https://appstoreconnect.apple.com/) with the same bundle identifier.
2. In Xcode: **Product → Archive → Distribute App → TestFlight & App Store**.
3. Add yourself, then friends, as testers. See Apple's [TestFlight overview](https://developer.apple.com/testflight/).

## When something breaks

| Symptom | Usual cause |
|---|---|
| Black screen instead of camera | ARKit not ticked under XR Plug-in Management, or no camera usage description |
| "Untrusted Developer" on the phone | Trust the certificate under Settings → General → VPN & Device Management |
| Signing errors in Xcode | Bundle identifier already taken, or no team selected |
| Safari will not show the camera (Stage 0) | Page is served over HTTP rather than HTTPS |
| Huge repo, slow pushes | `Library/` was committed, or LFS was not installed before adding assets |
