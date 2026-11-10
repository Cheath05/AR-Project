# Stage 1 — Foundations

**Scheduled:** see [docs/schedule.md](../../docs/schedule.md) · **Tools:** Unity 6, Xcode, your iPhone · **No AR yet**

> **New to coding?** Work through [GUIDE.md](GUIDE.md) alongside this brief. It says which files to create, where the code goes and how to check each step.

## What you will learn

- **C# basics:** types, classes, methods and how they differ from the language you already know.
- **The Unity model:** scenes, GameObjects, components, and scripts that run once per frame.
- **The build loop:** Unity → Xcode → iPhone, and how signing fits in.
- **Git for Unity:** which folders to commit, and why large files go through LFS.

There is no AR in this stage. It exists so that in Stage 2 the only new thing is the AR.

## Read first

- A tour of C#: https://learn.microsoft.com/en-us/dotnet/csharp/tour-of-csharp/
- Unity Essentials pathway: https://learn.unity.com/pathway/unity-essentials
- Unity's iOS getting started guide: https://docs.unity3d.com/Manual/iphone-GettingStarted.html
- This repo's [setup guide](../../docs/setup.md), sections 1, 2, 4 and 5.

## Core build: hello, phone

**Goal:** a Unity scene with a script you wrote, running on your iPhone and committed to this repo.

There is no starter. Create the Unity project inside this folder (for example `projects/01-hello-phone/HelloPhone/`).

### Tasks

1. **Take the C# tour.** Read it with an editor open and type the examples. Note three things that differ from the language you know best. Which of them would have tripped you up?
2. **Install the tools.** Follow `docs/setup.md` for Xcode and Unity 6 with iOS Build Support. Create an empty project and press Play. What is the difference between the Scene view and the Game view?
3. **Work through Unity Essentials.** It is long; it is scheduled across two sessions. You are done when you can explain what a GameObject, a component, a prefab and a Transform are without looking them up.
4. **Write your first script.** Make an object in your own scene do something every frame, and something different when you touch the screen. Before you run it, predict: what happens to the speed of your motion on a phone running at 30 frames per second versus 60? Then find out what Unity gives you to fix that.
5. **Build to your iPhone.** Follow `docs/setup.md` section 4. Get your scene running on the phone, signed with your developer team. Does your touch code from task 4 behave the same as mouse clicks did in the editor?
6. **Commit it properly.** Install Git LFS, add a texture or model to the project and commit. Check that `Library/` is not in the commit and that the large file went through LFS. How can you tell from the command line?

### Done when

- [ ] A scene with your own script runs on your iPhone.
- [ ] It responds to touch.
- [ ] The project is committed with `Assets/`, `Packages/` and `ProjectSettings/` and nothing from `Library/`.
- [ ] `git lfs ls-files` lists at least one file.

### Check your understanding

Answer these in your own words in `docs/log.md`:

1. What is the difference between `Start` and `Update`? Name one thing that belongs in each.
2. A component is a class. What does Unity do with it that a plain C# class does not get?
3. Why does every asset have a `.meta` file, and what breaks if you do not commit them?
4. Unity builds an Xcode project, not an app. Why the extra step?

## Play quest: Constellation finder

**Scheduled:** backlog. A good first project for a free day over winter break.

**Goal:** hold the phone up to the sky and see star names in the direction you are pointing. No camera tracking, only the phone's sensors.

- The phone knows which way it is pointing from its gyroscope and compass. Research Unity's `Gyroscope` and `Compass` input: https://docs.unity3d.com/ScriptReference/Gyroscope.html
- Rotate a Unity camera to match the phone. The phone's axes and Unity's axes do not agree. Work out the conversion by experiment: which way does each axis point when the phone is flat on a table?
- Place a handful of bright stars on a large sphere around the camera. Star positions are published as right ascension and declination. How do you turn those two angles into a 3D direction?
- Label each star so the text always faces you.
- Stretch: account for your latitude and the time of day so the sky is correct. Stretch further: draw constellation lines.

This is device pose from sensors alone: rotation only, no position. Notice what it cannot do. Stage 2 adds the camera and gets position as well.

## If you get stuck

Ask for help in this order: concept explanation → hint → code review → minimal example. See [docs/roadmap.md](../../docs/roadmap.md#getting-help-without-getting-the-answer).
