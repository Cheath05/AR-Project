# Stage 1 — How to start

The [brief](README.md) says *what* to build. This guide says *how to go about it*: what to learn in which order, where files go, what to click in Unity, and how to tell each step worked. It does not contain the finished code.

This is the longest guide because it is where you learn C# and the Unity editor. Every later stage assumes it.

## Before you write anything

1. Finish Stage 0. The ideas (variables, functions, `if`, objects and dots, callbacks) are the same in C#; only the spelling changes.
2. Reread "How a Unity project is put together" in [docs/coding-basics.md](../../docs/coding-basics.md). It is the map for this whole stage.
3. Have [docs/setup.md](../../docs/setup.md) open. This guide points to it for installs and does not repeat them.

## New coding ideas in this stage

| Idea | Where you meet it | Learn it here |
|---|---|---|
| Types: `int`, `float`, `string`, `bool` | Every line of C# | https://learn.microsoft.com/en-us/dotnet/csharp/tour-of-csharp/tutorials/numbers-in-csharp |
| `if`, `while`, `for` | Task 1, then everywhere | https://learn.microsoft.com/en-us/dotnet/csharp/tour-of-csharp/tutorials/branches-and-loops |
| Lists | Task 1; used heavily from Stage 2 | https://learn.microsoft.com/en-us/dotnet/csharp/tour-of-csharp/tutorials/list-collection |
| Classes (a blueprint with data and methods) | Every Unity script is one | https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/tutorials/classes |
| `MonoBehaviour`, `Start`, `Update` | Task 4 | https://docs.unity3d.com/Manual/class-MonoBehaviour.html |
| `[SerializeField]` (a setting you can edit in the Inspector) | Task 4 | https://docs.unity3d.com/ScriptReference/SerializeField.html |
| `Debug.Log` (print to the Console) | Task 4 | https://docs.unity3d.com/ScriptReference/Debug.Log.html |
| `Time.deltaTime` (seconds since the last frame) | Task 4 | https://docs.unity3d.com/ScriptReference/Time-deltaTime.html |
| Touch input | Task 4 | https://docs.unity3d.com/Packages/com.unity.inputsystem@1.11/manual/Touch.html |

## Files you will add

By the end of the stage this folder looks like this. Unity creates most of it; you create the two marked files.

```
01-hello-phone/
├── README.md
├── GUIDE.md
└── HelloPhone/                  (the Unity project; Unity makes this in task 2)
    ├── Assets/
    │   ├── Scenes/
    │   │   └── Main.unity       (your scene)
    │   ├── Scripts/
    │   │   └── Spinner.cs       (you write this in task 4)
    │   └── Textures/
    │       └── (any image)      (you add this in task 6)
    ├── Packages/
    ├── ProjectSettings/
    ├── Library/                 (huge, auto-generated, never committed)
    └── iOSBuild/                (the Xcode project from task 5, never committed)
```

Two rules that hold for every stage from here on:

- **Scripts live in `Assets/Scripts/`.** One class per file.
- **The file name must equal the class name.** `Spinner.cs` contains `class Spinner`. If they differ, Unity will not let you attach the script.

## Task by task

### Task 1 — Learn enough C# (about 4 hours)

No Unity yet. These tutorials run in the browser with nothing to install. Do them in this order and **type every example yourself**:

1. Hello world: https://learn.microsoft.com/en-us/dotnet/csharp/tour-of-csharp/tutorials/hello-world
2. Numbers: https://learn.microsoft.com/en-us/dotnet/csharp/tour-of-csharp/tutorials/numbers-in-csharp
3. Tuples and types: https://learn.microsoft.com/en-us/dotnet/csharp/tour-of-csharp/tutorials/tuples-and-types
4. Branches and loops: https://learn.microsoft.com/en-us/dotnet/csharp/tour-of-csharp/tutorials/branches-and-loops
5. Lists: https://learn.microsoft.com/en-us/dotnet/csharp/tour-of-csharp/tutorials/list-collection

Then read (do not try to memorise) the classes tutorial: https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/tutorials/classes. You only need to come away knowing that a class has **fields** (its data) and **methods** (its functions).

If the text tutorials are not landing, watch the first handful of episodes of the video series instead: https://learn.microsoft.com/en-us/shows/csharp-fundamentals-for-absolute-beginners/

**What to notice coming from JavaScript:**

| JavaScript | C# |
|---|---|
| `const speed = 2;` | `float speed = 2f;` (you state the type; decimals for `float` end in `f`) |
| `function grow(amount) { }` | `void Grow(float amount) { }` (inputs have types; `void` means "returns nothing") |
| `console.log(x)` | `Debug.Log(x)` in Unity |
| Semicolons optional | Semicolons required |
| Loose code in a file | Everything lives inside a class |

**Check:** without looking, write on paper a method that takes two numbers and returns the bigger one. If you can, move on. If not, redo tutorial 4.

### Task 2 — Install the tools and make the project

1. Follow [docs/setup.md](../../docs/setup.md) sections 1 and 2 (Xcode, then Unity 6 with **iOS Build Support**). These are big downloads; start them and do task 1 while they run.
2. In Unity Hub: **Projects → New project → Universal 3D**. Set **Project name** to `HelloPhone` and **Location** to this folder (`AR-Project/projects/01-hello-phone`). Click **Create project**.
3. Wait. The first open takes several minutes.

**The editor layout.** Find each of these panels before going further. Overview: https://docs.unity3d.com/Manual/unity-editor.html

| Panel | Usually where | What it is |
|---|---|---|
| **Hierarchy** | Left | A list of every GameObject in the open scene |
| **Scene view** | Centre tab | A free camera for you to build with. Players never see this |
| **Game view** | Centre tab, next to Scene | What the app's camera sees. This is what ends up on the phone |
| **Inspector** | Right | The components and settings of whatever you selected |
| **Project** | Bottom | The files in your `Assets` folder |
| **Console** | Bottom tab, next to Project | Printed messages and errors. If you cannot see it: **Window → General → Console** |
| **Play button** | Top centre | Runs the scene inside the editor |

Moving around the Scene view: hold right mouse button and use `W A S D` to fly; scroll to zoom; select an object and press `F` to jump to it.

**Check:** press Play. The Game view shows an empty sky. Press Play again to stop.

> **The mistake everyone makes once:** changes you make *while Play is on* are thrown away when you stop. Always stop Play before editing.

### Task 3 — Unity Essentials

Do the pathway: https://learn.unity.com/pathway/unity-essentials. It is scheduled over two sessions. Do it in its own practice project as the pathway instructs, not in `HelloPhone`.

It covers more than you need. The parts that matter for this roadmap are the editor basics, 3D objects, and the programming mission. Skim audio and 2D if you are short on time.

**Check:** explain these four out loud, with no notes. If you can, you are done.

- **GameObject:** a thing in the scene.
- **Component:** a behaviour attached to a GameObject.
- **Transform:** the component every GameObject has, holding its position, rotation and scale.
- **Prefab:** a saved GameObject you can make copies of.

### Task 4 — Your first script

**Plain English:** make a cube. Every frame, turn it a little. When the screen is touched, do something different, such as change its colour or make it jump.

**Set up the scene (no code):**

1. Open `HelloPhone`. In the **Project** panel open `Assets/Scenes`. If the scene there is called `SampleScene`, right-click it → **Rename** → `Main`.
2. In the **Hierarchy**, right-click empty space → **3D Object → Cube**.
3. Select **Main Camera** in the Hierarchy, then look at the Game view. If you cannot see the cube, select the cube and in the Inspector set its **Transform → Position** to `0, 0, 0`, and set the camera's position to `0, 1, -5`.
4. Save with `Cmd + S`.

**Create the script:**

1. In the **Project** panel, right-click `Assets` → **Create → Folder**, name it `Scripts`.
2. Right-click `Scripts` → **Create → Scripting → MonoBehaviour Script**. Type the name `Spinner` straight away, before clicking anywhere else, and press Return. (The same menu is at **Assets → Create → Scripting** in the top menu bar.)
3. Double-click `Spinner`. It opens in VS Code. If it opens in something else, set **Unity → Settings → External Tools → External Script Editor** to Visual Studio Code.

**Attach it:** drag `Spinner` from the Project panel onto **Cube** in the Hierarchy. Select the cube: the Inspector now shows a **Spinner (Script)** component at the bottom. A script that is not attached to anything never runs.

**The skeleton.** Replace the file's contents with this, then fill in the bodies yourself:

```csharp
using UnityEngine;

public class Spinner : MonoBehaviour
{
    // A setting you can change in the Inspector without touching code.
    [SerializeField] float degreesPerSecond = 90f;

    // Unity calls this once, when the scene starts.
    void Start()
    {
        // 1. Print a message so you can see that the script is running.
    }

    // Unity calls this every frame.
    void Update()
    {
        // 2. Turn this object a little around the up axis.
        //    How much? degreesPerSecond, scaled by how long this frame took.

        // 3. If the screen was touched or the mouse was clicked this frame,
        //    do something different.
    }
}
```

**Step 1 is one line.** `Debug.Log("some text");` prints to the Console.

**Step 2 is one line.** Every script can reach its own GameObject's Transform with the word `transform`. Transforms have a `Rotate` method: https://docs.unity3d.com/ScriptReference/Transform.Rotate.html. It takes three numbers: degrees around x, y and z.

First write it with a fixed number, such as 1 degree around y, and press Play. Then answer the brief's question: `Update` runs once per frame, so what happens on a phone running at half the frame rate? The fix is `Time.deltaTime`, the number of seconds the last frame took. Work out why *multiplying* by it turns "degrees per frame" into "degrees per second".

**Step 3 is an `if`.** You need to ask "was there a press this frame?" The simplest way that works for both the editor's mouse and the phone's touchscreen uses the Input System's `Pointer`:

```
using UnityEngine.InputSystem;        // goes at the top of the file, under the other using line
```

```
if (Pointer.current != null && Pointer.current.press.wasPressedThisFrame)
{
    // what to do on a tap
}
```

`Pointer.current` is whichever pointing device is in use: the mouse in the editor, your finger on the phone. Stage 2 switches to the fuller `EnhancedTouch` API, which can track several fingers.

Ideas for "something different": reverse the spin by flipping the sign of `degreesPerSecond`; move the cube up using `transform.position` (https://docs.unity3d.com/ScriptReference/Transform-position.html); or change its scale with `transform.localScale`. Pick one and look up how.

**Check:**

- [ ] Press Play. Your message appears in the **Console**.
- [ ] The cube spins.
- [ ] Select the cube while playing and change **Degrees Per Second** in the Inspector. The speed changes live.
- [ ] Clicking in the Game view triggers your "something different".

**If it fails:**

| What you see | Likely cause |
|---|---|
| Red error in the Console, Play will not start | A compile error. Read it: it gives the file and line number, like `Spinner.cs(14,9)`. Usually a missing `;` or `}` |
| "Can't add script component… the script class cannot be found" | The file name and the class name differ. Both must be exactly `Spinner` |
| Nothing prints, nothing moves | The script is not attached to the cube |
| The Inspector field is missing | `[SerializeField]` is misspelled, or there is a compile error elsewhere |
| `The name 'Pointer' does not exist` | The `using UnityEngine.InputSystem;` line is missing |
| `InvalidOperationException` mentioning `UnityEngine.Input` | You used the old `Input.` class. New Unity 6 projects use the Input System package; use `Pointer` as above |
| The cube spins madly | You have not multiplied by `Time.deltaTime` yet |

Commit when all four checks pass.

### Task 5 — Build to your iPhone

Follow [docs/setup.md](../../docs/setup.md) section 4 exactly. The short version:

1. **File → Build Profiles → iOS → Switch Platform.** Wait for it to finish.
2. **Edit → Project Settings → Player → iOS tab → Other Settings → Bundle Identifier:** set something unique, such as `com.yourname.hellophone`.
3. In Build Profiles, make sure your `Main` scene is in the scene list, then click **Build**. When asked where, create a folder named `iOSBuild` inside `HelloPhone`.
4. Open `iOSBuild/Unity-iPhone.xcodeproj`. Xcode opens.
5. Click the blue **Unity-iPhone** item at the top of the left sidebar → the **Unity-iPhone** target → **Signing & Capabilities** → tick **Automatically manage signing** → choose your **Team**.
6. Plug in the iPhone and unlock it. Pick it in the device menu at the top of the Xcode window. Press the Play (▶) button.

**Check:** the cube spins on your phone, and tapping the screen does your "something different".

**If it fails:** see the "When something breaks" table at the bottom of `docs/setup.md`. First builds usually fail on signing (no team selected) or on the phone not having **Developer Mode** switched on.

This is the loop for the rest of the roadmap: change something in Unity → Build → Run from Xcode. After the first time, use **Build And Run** in Unity to do both steps at once.

### Task 6 — Commit it properly

1. Install Git LFS once: [docs/setup.md](../../docs/setup.md) section 5.
2. Add any image to the project: in the Project panel create `Assets/Textures` and drag a `.png` or `.jpg` into it from Finder.
3. In the VS Code terminal, from the repo root:

```bash
git status
```

Look at the list. You should see paths under `HelloPhone/Assets`, `HelloPhone/Packages` and `HelloPhone/ProjectSettings`. You should **not** see `Library`, `Temp`, `Logs`, `UserSettings` or `iOSBuild`. If you do, stop and ask; the `.gitignore` is not being applied.

```bash
git add -A
```

```bash
git commit -m "Stage 1: spinning cube runs on iPhone"
```

```bash
git lfs ls-files
```

That last command lists files stored through LFS. Your image should be on it.

```bash
git push
```

**Check:** on GitHub, open the image file in the repo. It shows "Stored with Git LFS".

Every asset in Unity has a matching `.meta` file. Always commit them. The brief's "Check your understanding" asks why.

## Constellation finder: how to start

A backlog quest. Do it in a new scene inside `HelloPhone` (**File → New Scene**, save as `Assets/Scenes/Sky.unity`).

**Script:** `Assets/Scripts/SkyCamera.cs`, attached to **Main Camera**.

```csharp
using UnityEngine;

public class SkyCamera : MonoBehaviour
{
    void Start()
    {
        // 1. Switch the gyroscope on. It is off by default.
    }

    void Update()
    {
        // 2. Read the phone's current rotation from the gyroscope.
        // 3. Convert it from the phone's axes to Unity's axes.
        // 4. Apply it to this camera's rotation.
    }
}
```

- The gyroscope reference, including how to enable it and read its `attitude`: https://docs.unity3d.com/ScriptReference/Gyroscope.html. Note it is part of the older `Input` class; check that page and the Input System docs for what your project's input setting allows, and ask if the two disagree.
- Sensors do not work in the editor. You must build to the phone to test anything, so use `Debug.Log` less and an on-screen text label more.
- Build order: one cube in front of the camera and one behind → turning the phone turns the view → fix the axes until "up" is up → replace cubes with a few stars → add labels.
- Stars: a second script, `StarField.cs`, holding a list of names and two angles each, which creates a small sphere per star in `Start`. Get one star working before making a list.

## Committing your work

After each task that changes the project, from the repo root:

```bash
git add -A
```

```bash
git commit -m "Stage 1: task 4, cube spins and reacts to tap"
```

```bash
git push
```
