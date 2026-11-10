# Stage 2 — How to start

The [brief](README.md) says *what* to build. This guide says *how to go about it*: which objects to add to the scene, which scripts to create, which method each piece of code goes in, and how to tell each step worked. It does not contain the finished code.

## Before you write anything

1. Finish Stage 1. You should be able to create a script, attach it, expose a field in the Inspector, read the Console and build to the phone without looking anything up.
2. AR does not run when you press Play in the editor: there is no camera feed or room. **Every check in this guide happens on the phone.** A build takes a few minutes, so change one thing per build.
3. The phone has no Console. Task 1 adds an on-screen label so you can still see your `Debug.Log`-style messages.

## New coding ideas in this stage

| Idea | Where you meet it | Learn it here |
|---|---|---|
| `List<T>` (a growable list of things) | Task 3: the raycast fills one | https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.list-1 |
| A method that returns `bool` and fills a list you pass in | Task 3 | https://docs.unity3d.com/Packages/com.unity.xr.arfoundation@6.0/manual/features/raycasts/single-raycasts.html |
| `Pose` (a position and a rotation together) | Task 3, 4 | Same page |
| `Instantiate` (make a copy of a prefab) | Task 4 | https://docs.unity3d.com/Manual/InstantiatingPrefabs.html |
| `Destroy` | Task 6 | https://docs.unity3d.com/ScriptReference/Object.Destroy.html |
| Enhanced touch (several fingers, phases) | Task 3, 5 | https://docs.unity3d.com/Packages/com.unity.inputsystem@1.11/manual/Touch.html |
| Physics raycast (hitting your own objects) | Task 5 | https://docs.unity3d.com/ScriptReference/Physics.Raycast.html |
| UI buttons calling your methods | Task 6 | https://docs.unity3d.com/Packages/com.unity.ugui@2.0/manual/script-Button.html |

## Files you will add

```
02-tap-to-place/
├── README.md
├── GUIDE.md
└── TapToPlace/                      (Unity project)
    └── Assets/
        ├── Scenes/
        │   └── Main.unity
        ├── Prefabs/
        │   ├── PlaneVisual.prefab   (task 2)
        │   └── PlacedCube.prefab    (task 4)
        └── Scripts/
            ├── StatusLabel.cs       (task 1: messages on screen)
            ├── TapToPlace.cs        (tasks 3–4)
            ├── ObjectMover.cs       (task 5)
            └── PlaneToggle.cs       (task 6)
```

Small scripts with one job each are easier to debug than one big one.

## Task by task

### Task 1 — An AR scene on your phone

No AR code yet. This is all editor setup.

**Project:**

1. Unity Hub → **New project → Universal 3D**, name `TapToPlace`, location this folder.
2. Follow [docs/setup.md](../../docs/setup.md) section 3: install **AR Foundation** and **Apple ARKit XR Plugin**, tick **ARKit** under **XR Plug-in Management → iOS**, and set the **Camera Usage Description** and **Bundle Identifier**. Unity's own page for the ARKit settings: https://docs.unity3d.com/Packages/com.unity.xr.arkit@6.0/manual/project-configuration-arkit.html
3. **File → Build Profiles → iOS → Switch Platform.**

**Scene** (from the AR Foundation manual: https://docs.unity3d.com/Packages/com.unity.xr.arfoundation@6.0/manual/project-setup/scene-setup.html):

1. Rename the scene `Main`.
2. In the **Hierarchy**, delete the default **Main Camera**. The XR Origin brings its own.
3. Right-click in the Hierarchy → **XR → AR Session**.
4. Right-click in the Hierarchy → **XR → XR Origin (Mobile AR)**. Expand it: it contains **Camera Offset → Main Camera**.

Those are the two objects the brief asks about. Work out from the manual what each is for.

**On-screen status label.** You need this in every AR stage.

1. Right-click in the Hierarchy → **UI → Text - TextMeshPro**. If Unity offers to import TMP Essentials, accept. A **Canvas** and an **EventSystem** appear too.
2. Select the text object, rename it `Status`, and in the Inspector anchor it to the bottom of the screen and make the font large enough to read on a phone.
3. Create `Assets/Scripts/StatusLabel.cs` and attach it to the `Status` object.

```csharp
using TMPro;
using UnityEngine;

public class StatusLabel : MonoBehaviour
{
    // Lets any other script write `StatusLabel.Show("text")`.
    static StatusLabel instance;

    TMP_Text label;

    void Awake()
    {
        // 1. Remember this object in `instance`.
        // 2. Find the TMP_Text component on this GameObject and store it in `label`.
    }

    public static void Show(string message)
    {
        // 3. Put `message` into the label's text.
        // 4. Also send it to Debug.Log, so it shows in Xcode's console.
    }
}
```

New here: `Awake` runs even earlier than `Start`; `GetComponent<TMP_Text>()` fetches a component from the same GameObject; `static` means "belongs to the class, not to one object", which is what lets other scripts call `StatusLabel.Show(...)` without a reference.

**Check:** build to the phone. You see the camera feed with your label over it. Call `StatusLabel.Show("hello")` from a `Start` method somewhere to prove the label works.

**If it fails:**

| What you see | Likely cause |
|---|---|
| Black screen | **ARKit** not ticked under XR Plug-in Management, or no Camera Usage Description |
| App closes at once | Camera permission was denied; delete the app and reinstall |
| Normal Unity sky, no camera feed | You are missing the AR Session or XR Origin, or kept the old Main Camera |
| Label missing | It is off screen. Check its position on the Canvas in the Game view |

Commit.

### Task 2 — See the planes

No code.

1. Select **XR Origin** in the Hierarchy. In the Inspector: **Add Component → AR Plane Manager**. (The manual: https://docs.unity3d.com/Packages/com.unity.xr.arfoundation@6.0/manual/features/plane-detection/arplanemanager.html)
2. It has a **Plane Prefab** slot. The manager makes one copy of that prefab for every plane it finds. You need to give it one:
   - Try right-clicking in the Hierarchy → **XR → AR Default Plane**. If your editor has that entry, it creates a ready-made plane visual. Check the manual for the exact name if it is not there.
   - Otherwise, copy the **AR Plane Debug Visualizer** prefab from the AR Foundation samples repo, which the manual recommends.
3. Create the folder `Assets/Prefabs`. Drag the plane object from the Hierarchy into it to make a prefab, rename the prefab `PlaneVisual`, then delete the object from the Hierarchy.
4. Select **XR Origin** again and drag `PlaneVisual` from the Project panel into the **Plane Prefab** slot.
5. Leave **Detection Mode** on Everything for now.

**Check:** build. Point the phone at the floor and move slowly side to side. Patches appear and grow. Try a table, a wall, a plain white wall, a glossy surface. Write down what you see; the brief asks.

Commit.

### Task 3 — Cast a ray

**Plain English:** every frame, check if a finger just touched the screen. If so, ask AR Foundation whether that screen point lands on a detected plane. If it does, show the position.

**Setup:** select **XR Origin** → **Add Component → AR Raycast Manager**.

**Script:** `Assets/Scripts/TapToPlace.cs`. Attach it to **XR Origin**. After attaching, drag the **XR Origin** object itself into the script's **Raycast Manager** slot in the Inspector (Unity picks the right component off it).

```csharp
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.InputSystem.EnhancedTouch;
using UnityEngine.XR.ARFoundation;
using UnityEngine.XR.ARSubsystems;
using Touch = UnityEngine.InputSystem.EnhancedTouch.Touch;

public class TapToPlace : MonoBehaviour
{
    [SerializeField] ARRaycastManager raycastManager;
    [SerializeField] GameObject objectPrefab;        // used from task 4

    // The raycast writes its results into this list. Reuse one list; do not make a new one each frame.
    readonly List<ARRaycastHit> hits = new List<ARRaycastHit>();

    void OnEnable()
    {
        // 1. Switch enhanced touch on.
    }

    void OnDisable()
    {
        // 2. Switch enhanced touch off.
    }

    void Update()
    {
        // 3. If there are no active touches, stop here (return).
        // 4. Take the first touch. If its phase is not "Began", stop here.
        //    (Otherwise you would place an object on every frame the finger is down.)
        // 5. Raycast from the touch's screen position against planes.
        // 6. If it hit, the nearest hit is first in the list. Show its position.
        // 7. (Task 4) Create the object there.
    }
}
```

**What each step looks like:**

- **Steps 1–2:** one call each on `EnhancedTouchSupport`. The Touch page linked above shows it.
- **Step 3:** `Touch.activeTouches` is a list; lists have a `Count`.
- **Step 4:** a touch has a `phase`; compare it with `UnityEngine.InputSystem.TouchPhase.Began`.
- **Step 5:** `raycastManager.Raycast(...)` takes three things: a screen position (the touch has a `screenPosition`), the `hits` list, and which kind of trackable to test against. For "inside the detected outline of a plane" the value is `TrackableType.PlaneWithinPolygon`. It returns `true` if anything was hit, so it goes inside an `if (...)`.
- **Step 6:** `hits[0]` is the nearest. It has a `pose`, and a pose has a `position` and a `rotation`.

The brief asks why this is not a physics raycast. Hint: a physics raycast only hits objects with colliders in your Unity scene. What is the floor?

**Check:** build. Tap the floor where a plane is showing. The label shows three numbers. Tap somewhere with no plane: nothing. The numbers are metres from where the phone was when the app started.

**If it fails:**

| What you see | Likely cause |
|---|---|
| `NullReferenceException` in Xcode's console | The **Raycast Manager** slot in the Inspector is empty |
| Never hits | No AR Raycast Manager on XR Origin, or you are tapping where no plane has been found yet |
| Fires continuously while the finger is down | Step 4 is missing |
| `'Touch' is an ambiguous reference` | The `using Touch = ...` line at the top is missing |
| Nothing at all | Enhanced touch was never enabled (step 1) |

Commit.

### Task 4 — Place an object

**Plain English:** at the hit, make a copy of a cube prefab, with the hit's position and rotation.

**Make the prefab (no code):**

1. Hierarchy → **3D Object → Cube**. Before you set its size, write down your prediction for the brief's question: how big is a cube of scale 1 in your room?
2. Set its **Scale** to `0.1, 0.1, 0.1`.
3. A cube's origin is its centre, so it would sit half-sunk into the floor. Fix it properly: create an empty GameObject named `PlacedCube`, make the cube its child, and move the *child* up by half its height (`0.05` on Y). Now the parent's origin is the cube's base.
4. Drag `PlacedCube` into `Assets/Prefabs`, then delete it from the Hierarchy.
5. Select **XR Origin** and drag the `PlacedCube` prefab into your script's **Object Prefab** slot.

**Code:** step 7 in `Update` is one line. `Instantiate` takes a prefab, a position and a rotation. See https://docs.unity3d.com/ScriptReference/Object.Instantiate.html. Keep what it returns in a variable; task 5 needs it.

**Check:** build. Tap the floor: a cube appears, resting on the surface. Walk around it. Place one on a table and one on a wall. Which way does the wall cube point, and why? (Look at what the pose's rotation means for a vertical plane.)

**If it fails:**

| What you see | Likely cause |
|---|---|
| Nothing appears | **Object Prefab** slot is empty, or the cube is 1 metre wide and you are standing inside it |
| Cube is half in the floor | You skipped the parent/child offset in step 3 |
| Cube floats or drifts | Normal at first; move the phone around more so tracking improves. Stage 3 deals with drift |

Commit.

### Task 5 — Move and rotate

**Plain English:** if a touch starts on an existing object, that object becomes "selected" and follows the finger along the plane. If two fingers are down, twist the selected object. If a touch starts on empty floor, place a new object as before.

This is the hardest task in the stage. Build it in four small steps, with a build between each.

**Setup:** your cube needs a collider to be hit by a physics ray. Unity's cube has a **Box Collider** already; check it is still on the child cube inside the prefab.

**Script:** `Assets/Scripts/ObjectMover.cs`, attached to **XR Origin**, with slots for the raycast manager and the camera (drag **Main Camera** from under XR Origin).

```csharp
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.XR.ARFoundation;
using UnityEngine.XR.ARSubsystems;
using Touch = UnityEngine.InputSystem.EnhancedTouch.Touch;

public class ObjectMover : MonoBehaviour
{
    [SerializeField] ARRaycastManager raycastManager;
    [SerializeField] Camera arCamera;

    readonly List<ARRaycastHit> hits = new List<ARRaycastHit>();

    // The object being dragged right now, or null if none.
    GameObject selected;

    public bool HasSelection => selected != null;

    void Update()
    {
        // 1. No touches: clear `selected` and return.
        // 2. One touch that just began: physics-raycast from the camera through the touch.
        //    If it hits a placed object, store it in `selected`.
        // 3. One touch that is moving, and something is selected: AR-raycast against planes
        //    and move `selected` to the hit position.
        // 4. Two touches, and something is selected: work out how much the line between
        //    the two fingers has turned since last frame, and rotate `selected` by that.
        // 5. Touch ended: clear `selected`.
    }
}
```

**How to go about it:**

- **Step 2** uses two things: `arCamera.ScreenPointToRay(position)` turns a screen point into a ray (https://docs.unity3d.com/ScriptReference/Camera.ScreenPointToRay.html), and `Physics.Raycast(ray, out RaycastHit hit)` tells you what it struck. `hit.collider.gameObject` is the thing hit. That is the child cube; you want its parent. Look up `transform.root` or `transform.parent`.
- How do you know the thing hit is one of *your* placed objects and not something else? Simplest: give the prefab a **Tag** (Inspector, top) such as `Placed`, and check `CompareTag("Placed")`.
- **Step 3** is the same AR raycast as task 3, used to set `selected.transform.position`.
- **Step 4:** the direction from finger A to finger B is a 2D vector. Its angle this frame minus its angle last frame is how much to rotate. Look up `Mathf.Atan2` and `Vector2.SignedAngle`. Each touch also has a `delta` (movement since last frame), so last frame's position is `screenPosition - delta`.
- **The brief's question, "place or move?":** `TapToPlace` and `ObjectMover` both see the same touch. Decide who goes first. One approach: give `TapToPlace` a slot for the `ObjectMover`, and have it place only if the touch did not land on an existing object. Work out the order on paper before coding.

Build order: tap an object and the label says "selected" → drag moves it → placing still works on empty floor → two-finger rotate.

**If it fails:**

| What you see | Likely cause |
|---|---|
| Tapping an object places another on top of it | The "place or move" check is missing or runs in the wrong order |
| Object jumps to a different height when dragged | You moved the child, not the parent |
| Physics raycast never hits | The collider was removed, or you passed the wrong camera |
| Rotation spins wildly | Angle is in radians where degrees are expected, or you are adding the total angle instead of the change |

Commit after each of the four steps.

### Task 6 — Delete, and tidy up

**Delete.** Decide the gesture first. Easiest: a **Delete** button that removes the selected object. That means "selected" must survive the finger lifting, so change step 5 of `ObjectMover` so that a selection stays until you tap empty space. Add a public method:

```csharp
public void DeleteSelected()
{
    // 1. If nothing is selected, return.
    // 2. Destroy the selected GameObject.
    // 3. Clear `selected`.
}
```

**Buttons (no code):** Hierarchy → right-click **Canvas** → **UI → Button - TextMeshPro**. Select the button; in the Inspector find **On Click ()**, press **+**, drag **XR Origin** into the object slot, and choose **ObjectMover → DeleteSelected** from the dropdown. That is how a button calls your method.

A tap on a button is also a touch on the screen, so it will try to place a cube underneath. Look up `EventSystem.current.IsPointerOverGameObject` and use it to ignore touches that are on UI.

**Hide the planes.** `Assets/Scripts/PlaneToggle.cs`, attached to **XR Origin**, with a slot for the `ARPlaneManager`.

```csharp
using UnityEngine;
using UnityEngine.XR.ARFoundation;

public class PlaneToggle : MonoBehaviour
{
    [SerializeField] ARPlaneManager planeManager;

    bool visible = true;

    public void Toggle()
    {
        // 1. Flip `visible`.
        // 2. Show or hide every plane the manager currently has.
        // 3. Decide: should detection itself also stop? Try both and compare.
    }
}
```

- For step 2, the manager has a method that sets all its plane objects active or inactive in one call. Search the `ARPlaneManager` scripting reference for "SetTrackablesActive". The manual warns never to `Destroy` planes yourself.
- For step 3, disabling the manager component (`planeManager.enabled = false`) stops detection. Then answer the brief's question: with visuals hidden *and* detection off, does tap-to-place still work on planes already found? What about a new area of floor?
- Planes found *after* you hide the visuals will still appear. How could you handle that? (The manager has a `trackablesChanged` event that tells you when planes are added.)

**Check:** all four boxes in the brief's "Done when" list.

Commit.

## Quests: how to start

Make each quest a **new scene** in the same `TapToPlace` project (**File → New Scene**, then repeat the AR Session and XR Origin setup, or duplicate `Main.unity` and strip it back). You keep your prefabs and scripts.

### Exploded diagrams

- **Files:** `Assets/Models/` for the model, `Assets/Scripts/Exploder.cs`, `Assets/Scripts/PartLabel.cs`.
- Import a `.fbx` or `.glb` whose parts are separate objects. Expand it in the Hierarchy to check: each part should be its own child.
- `Exploder` goes on the model's root. In `Start`, loop over `transform` children and remember each one's starting local position in a `List<Vector3>`. A public method `SetAmount(float t)` moves each child to `start + direction * t`. The direction for each part: from the model's centre to that part.
- Add a **UI → Slider**; wire its **On Value Changed** to `Exploder.SetAmount` (choose the "dynamic float" entry at the top of the dropdown). Slider docs: https://docs.unity3d.com/Packages/com.unity.ugui@2.0/manual/script-Slider.html
- Labels: reuse the physics-raycast tap from task 5 to find which part was tapped; show its `name`. Each part needs a collider (**Add Component → Mesh Collider**).
- Build order: model placed on desk → slider explodes it → tap shows a name → quiz mode.

### Graph in your room

- **Files:** `Assets/Scripts/SurfaceGraph.cs` on an empty GameObject that also has a **Mesh Filter** and **Mesh Renderer**.
- A mesh is a list of points (`Vector3[] vertices`) and a list of triangles (`int[] triangles`, three indices per triangle). Reference: https://docs.unity3d.com/ScriptReference/Mesh.html
- Write one method `float F(float x, float z)` that returns the height. Write another, `Rebuild()`, with two nested `for` loops over a grid that fills the vertices, then a second pair of loops that fills the triangles (two per grid square).
- Build order: **in the editor, no AR**, get a flat 10×10 grid showing → make it follow `F` → then place it with tap-to-place → then sliders.

### Physics sandbox

- Your plane prefab needs a **Mesh Collider** so balls land on it. Check whether your `PlaneVisual` has one.
- Ball prefab: a small sphere with a **Rigidbody** component. `BallLauncher.cs` instantiates it at the camera and sets its velocity along `arCamera.transform.forward`.
- Gravity is a single global setting, `Physics.gravity`. Bounciness and friction live on a **Physics Material** asset assigned to the collider.

### Measuring tape

- `Assets/Scripts/MeasuringTape.cs`, with a **Line Renderer** on the same object.
- Keep two `Vector3` fields and a counter for which tap you are on. First tap sets point A, second sets point B, third starts again.
- `Vector3.Distance(a, b)` gives metres. Set the line's two positions to A and B.
- The label is a world-space TextMeshPro object placed at the midpoint; make it face the camera each frame with `transform.LookAt` (https://docs.unity3d.com/ScriptReference/Transform.LookAt.html).

### AR graffiti

- The camera under XR Origin moves with the phone, so `arCamera.transform.position` is the phone's position in the room each frame.
- One **Line Renderer** per stroke. While a finger is down, add a point when the camera has moved more than a centimetre from the last point.
- Keep finished strokes in a `List<GameObject>` so undo can destroy the last one.

### Desk tower defence / floor is lava

- Start on paper: the rules in five lines.
- Get the game working **in the editor on an ordinary Unity plane first**, with no AR. Then swap the fake plane for a detected one.
- An `ARPlane` has a `boundary` (its outline points) and a `center`. Start simpler: use only the centre and a fixed radius as the play area.

## Committing your work

After each task, from the repo root:

```bash
git add -A
```

```bash
git commit -m "Stage 2: task 4, cube placed on tapped plane"
```

```bash
git push
```
