# Setup

What to install and how to get a Unity scene onto your iPhone. You do this during Stage 1; Stage 0 needs only a browser and a text editor.

## Get the repo onto your Mac (do this first)

"Cloning" means downloading a copy of this repo that stays linked to GitHub, so you can save your work back to it.

### Which app does what

| App | What you use it for | When you need it |
|---|---|---|
| **Terminal** | Typing Git commands (clone, commit, push) | Now |
| **[VS Code](https://code.visualstudio.com/)** | Writing code: HTML/JavaScript in Stage 0, C# from Stage 1. This is your editor (IDE) | Now |
| **Unity** | Building the AR scenes | Stage 1 onward |
| **Xcode** | Only for sending a Unity build to your iPhone. You will not write code in it until Stage 8 | Stage 1 onward |

VS Code is the recommendation because it is free, handles every language in this roadmap, and Unity supports it officially. [Rider](https://www.jetbrains.com/rider/) is a more powerful C# editor that is free for non-commercial use; worth a look after Stage 2, not before.

### Step by step

1. **Open Terminal.** Press `Cmd + Space`, type `Terminal`, press Return.

2. **Check Git is installed.** Type this and press Return. If macOS offers to install "command line developer tools", accept and wait for it to finish.

   ```bash
   git --version
   ```

3. **Make a folder for your projects and go into it.** `mkdir` makes a folder, `cd` moves into it, and `~` means your home folder.

   ```bash
   mkdir -p ~/Developer && cd ~/Developer
   ```

4. **Clone the repo.** This creates `~/Developer/AR-Project`.

   ```bash
   git clone https://github.com/Cheath05/AR-Project.git
   ```

5. **Go into it.**

   ```bash
   cd AR-Project
   ```

6. **Install VS Code** from https://code.visualstudio.com/ and drag it to Applications. Open it, then **File → Open Folder…** and choose `Developer/AR-Project`. The file list on the left is the repo.

7. **Add two extensions** (the squares icon in VS Code's left bar, then search by name):
   - [Live Server](https://marketplace.visualstudio.com/items?itemName=ritwickdey.LiveServer), for Stage 0. Right-click `index.html` → **Open with Live Server** to see the page in your Mac's browser.
   - [Unity](https://marketplace.visualstudio.com/items?itemName=visualstudiotoolsforunity.vstuc), from Stage 1. It gives C# autocomplete and error checking for Unity code.

VS Code has a terminal built in (**Terminal → New Terminal**), already pointed at the repo folder. From here on you can use that one and skip the separate Terminal app.

### Saving your work (the loop you will repeat every session)

Run these from the repo folder.

See what you changed:

```bash
git status
```

Stage everything you changed, ready to be saved:

```bash
git add -A
```

Save a snapshot with a short description of what you did:

```bash
git commit -m "Stage 0: cube appears on card"
```

Upload it to GitHub:

```bash
git push
```

The first push may ask you to sign in to GitHub in the browser. If you ever work from a second computer, run `git pull` at the start of a session to download the latest version.

You are working alone, so committing straight to `main` is fine. Branches and pull requests are useful later, when you want to try something risky without breaking what works.

### If you would rather click than type

- **VS Code's Source Control panel** (the branching icon in the left bar) does status, add, commit and push with buttons. Guide: https://code.visualstudio.com/docs/sourcecontrol/overview
- **[GitHub Desktop](https://desktop.github.com/)** is a separate app that does the same, including cloning (**File → Clone Repository**).

Learning the four commands above is still worth it. Nearly every tutorial you follow will assume them. GitHub's plain-language intro: https://docs.github.com/en/get-started/using-git/about-git

## Stage 0 needs

- The repo cloned and open in VS Code (the section above).
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

Git LFS stores large files (models, textures, audio) outside the normal Git history so the repo stays fast. It installs through [Homebrew](https://brew.sh/), a tool for installing developer software on a Mac. If `brew --version` prints "command not found", install Homebrew first using the one-line command on its home page.

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
