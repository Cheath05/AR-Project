# Stage 7 — How to start

The [brief](README.md) says *what* to build. This guide says *how to go about it*: which files, which settings, in what order, and how to tell each step worked. It does not contain the finished code.

This is a short stage. Two scripts carry the core build.

## Before you write anything

1. Check your iPhone has Face ID. Face tracking uses the front depth camera; the [ARKit face tracking page](https://docs.unity3d.com/Packages/com.unity.xr.arkit@6.0/manual/arkit-face-tracking.html) lists the requirements.
2. Do the first units of Unity Learn's face filters course (link in the brief). It shows the editor side on video.
3. Start a **new** Unity project in this folder called `FaceMask`. Do not copy Stage 2: face tracking uses a different camera and none of the plane code applies.

## New coding ideas in this stage

| Idea | Where you meet it | Learn it here |
|---|---|---|
| Events with `+=` and `-=` (again, on a different manager) | Task 4 | https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/events/ |
| `enum` values as names for things | Task 4: blend shape names | https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/enum |
| `NativeArray`: a list borrowed from the phone that you must hand back | Task 4 | https://docs.unity3d.com/ScriptReference/Unity.Collections.NativeArray_1.html |
| `using (...) { }`: hand it back automatically | Task 4 | https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/using |
| Casting: treating a general thing as its specific type | Task 4 | Search "C# cast expression" on learn.microsoft.com |
| A `bool` that remembers last frame (state) | Task 5 | Covered below |
| Particle systems | Task 5 | https://docs.unity3d.com/Manual/ParticleSystems.html |

## Files you will add

```
07-face-mask/
└── FaceMask/
    └── Assets/
        ├── Materials/
        │   └── FaceWire.mat           (task 2)
        ├── Prefabs/
        │   └── FacePrefab.prefab      (tasks 2 and 3)
        └── Scripts/
            ├── BlendShapeReader.cs    (task 4)
            └── ExpressionTrigger.cs   (task 5)
```

## Task by task

### Task 1 — A face tracking project

No code. Four settings differ from every project so far.

1. Install **AR Foundation** and **Apple ARKit XR Plugin** and tick ARKit under **Project Settings → XR Plug-in Management → iOS**, as in [docs/setup.md](../../docs/setup.md).
2. **Project Settings → XR Plug-in Management → ARKit**: tick **Face Tracking**. This is off by default.
3. Add an **AR Session** and an **XR Origin** to the scene (under **GameObject → XR**).
4. Select **XR Origin → Camera Offset → Main Camera**. On **AR Camera Manager**, set **Facing Direction** to **User**. That selects the front camera.
5. Select the **XR Origin** itself → **Add Component → AR Face Manager**. Leave **Face Prefab** empty for now.
6. Set the camera usage description and bundle identifier, then build.

**Check:** the app opens showing you, through the front camera.

**If it fails:**

| What you see | Likely cause |
|---|---|
| Rear camera shows | Facing Direction is still World; or another manager in the scene (planes, images) needs the rear camera, so the session chose it. Remove those managers |
| Black screen | ARKit not ticked under XR Plug-in Management, or no camera usage description |
| Build error mentioning face tracking | The Face Tracking box in step 2 is not ticked |

The brief asks whether world and face tracking can run together. The ARKit page above has a section on this; read it and answer in your log.

Commit.

### Task 2 — Show the face mesh

No code.

1. Look in **GameObject → XR** for **AR Default Face** and add it. If your version has no such entry, build it by hand: an empty GameObject with **AR Face**, **AR Face Mesh Visualizer**, **Mesh Filter** and **Mesh Renderer**.
2. Make a material in `Assets/Materials/` called `FaceWire`. A bright, half-transparent colour is enough to see the shape.
3. Put it on the Mesh Renderer.
4. Drag the object into `Assets/Prefabs/` as `FacePrefab`, then delete it from the scene.
5. Select the XR Origin and drag `FacePrefab` into **AR Face Manager → Face Prefab**.

**Check:** a coloured mesh covers your face and follows it. Open and close your mouth and watch whether the mesh changes shape. To count the vertices, add a temporary script on the prefab that logs the length of the `ARFace` component's `vertices` in `Update`.

**If it fails:**

| What you see | Likely cause |
|---|---|
| No mesh | Face Prefab slot empty, or the prefab has no AR Face Mesh Visualizer |
| Pink mesh | The material's shader does not match your render pipeline. Make a new material from **Create → Material** |
| Mesh appears then freezes | Face left the camera view. The face's tracking state becomes limited; bring it back |

Commit.

### Task 3 — Attach something

No code, if you use the hierarchy.

**Plain English:** the face prefab moves with your head. Anything that is a *child* of it moves too. This is the anchor idea from Stage 0, again.

1. Double-click `FacePrefab` to open it for editing.
2. Add a child: **3D Object → Sphere**, scaled to about 0.03 (3 cm). This is a nose.
3. Move it with the **Position** fields until it sits on the nose. You cannot see your face in the editor, so this is trial and error: change a number, build, look. Start with small steps along Z (forward from the face).
4. Once the nose is right, replace it or add glasses or a hat the same way.

To skip the guesswork, read the mesh: the `ARFace` component exposes `vertices`, and each one is a point on the face in the prefab's own space. A temporary script can place a tiny marker at vertex number N so you can find which number is the nose tip. The [AR Face page](https://docs.unity3d.com/Packages/com.unity.xr.arfoundation@6.0/manual/features/face-tracking/arface.html) describes what the component offers; check its API page for eye positions as well.

**Check:** the object stays on your face as you turn left, right, up and down.

**If it fails:**

| What you see | Likely cause |
|---|---|
| Object floats in front of or inside your head | Wrong offset. Units are metres; a face is about 0.15 wide |
| Object stays still while the mesh moves | It is in the scene, not a child inside the prefab |
| Object is huge | Scale 1 is one metre |

Commit.

### Task 4 — Read the blend shapes

**Plain English:** for the tracked face, ask ARKit for its list of about fifty expression numbers, find the three you care about, and show them on screen.

Blend shapes are not in the general AR Foundation face API. They come from the ARKit-specific *face subsystem*. The pieces, all verified against the ARKit plug-in API pages:

- The class is `ARKitFaceSubsystem`, in the namespace `UnityEngine.XR.ARKit`.
- It has one method of its own: `GetBlendShapeCoefficients(TrackableId faceId, Allocator allocator)`, which returns a `NativeArray<ARKitBlendShapeCoefficient>`.
- You own that array and must call `Dispose` on it when done, or put it in a `using` block.
- Each entry pairs a location with an amount. The locations are the 52 names in the `ARKitBlendShapeLocation` enum, for example `JawOpen`, `EyeBlinkLeft`, `MouthSmileLeft`: https://docs.unity3d.com/Packages/com.unity.xr.arkit@6.0/api/UnityEngine.XR.ARKit.ARKitBlendShapeLocation.html
- Open the `ARKitBlendShapeCoefficient` page from there for the exact names of its two fields.

The face manager holds a general face subsystem in its `subsystem` property. On an iPhone that object *is* an `ARKitFaceSubsystem`; you cast it to reach the extra method. The **Blend shapes** scene in the AR Foundation samples does exactly this and is worth reading beside your own file.

`BlendShapeReader.cs`, on the **root of `FacePrefab`** (so there is one per face):

```csharp
using System;
using Unity.Collections;
using UnityEngine;
using UnityEngine.XR.ARFoundation;
using UnityEngine.XR.ARKit;

public class BlendShapeReader : MonoBehaviour
{
    public float JawOpen { get; private set; }
    public float BlinkLeft { get; private set; }
    public float SmileLeft { get; private set; }

    private ARFace face;
    private ARKitFaceSubsystem faceSubsystem;

    void Awake()
    {
        // 1. Get the ARFace component on this GameObject
        // 2. Find the ARFaceManager in the scene (a prefab cannot have it dragged in)
        // 3. Cast its subsystem to ARKitFaceSubsystem and store it
    }

    void OnEnable()
    {
        // listen to face.updated
    }

    void OnDisable()
    {
        // stop listening
    }

    void OnFaceUpdated(ARFaceUpdatedEventArgs args)
    {
        // 1. If faceSubsystem is null, return (this happens in the editor)
        // 2. Ask it for the coefficients of face.trackableId, with Allocator.Temp,
        //    inside a using block
        // 3. Loop over them
        // 4. If an entry's location is JawOpen, store its amount in JawOpen
        // 5. Same for the other two
    }
}
```

To display them, add a Canvas with a TextMeshPro text and a tiny script that finds the reader each frame and writes the three numbers to the text. Format them to two decimal places so they do not jitter unreadably.

**Check:** three numbers on screen. Opening your mouth moves the first from near 0 toward 1. One caution: the front camera view is mirrored, so "left" may be the opposite of what you expect. Wink and see which number moves.

**If it fails:**

| What you see | Likely cause |
|---|---|
| Namespace `ARKit` not found | The Apple ARKit XR Plugin package is not installed |
| Numbers stay 0 | The cast gave null (you are in the editor, or step 3 ran before the manager started); or the names being compared are wrong |
| Console error about a leaked or undisposed NativeArray | The array was not disposed. Use the `using` form |
| `NullReferenceException` in `Awake` | The script is on a child, not on the object that has the AR Face component |

Commit.

### Task 5 — Trigger on an expression

**Plain English:** when jaw-open goes above a threshold, fire the effect once. Do not fire again until the mouth has closed.

The whole problem is turning a wobbly number into one clean event. You need to remember one fact between frames: *was it already open?*

`ExpressionTrigger.cs`, also on the root of `FacePrefab`:

```csharp
using UnityEngine;

public class ExpressionTrigger : MonoBehaviour
{
    [SerializeField] private BlendShapeReader reader;
    [SerializeField] private float openAbove = 0.6f;
    [SerializeField] private float closedBelow = 0.3f;
    [SerializeField] private ParticleSystem effect;

    private bool isOpen;

    void Update()
    {
        // 1. Read reader.JawOpen
        // 2. If we are NOT open and the value is above openAbove:
        //      set isOpen to true and fire the effect
        // 3. If we ARE open and the value is below closedBelow:
        //      set isOpen to false
    }
}
```

Why two thresholds and not one: at a single threshold the value flickers across the line and fires repeatedly. A gap between "open" and "closed" stops that. Because the thresholds are `[SerializeField]`, you can tune them in the Inspector.

For the effect: add a **Particle System** as a child in the prefab, untick **Play On Awake**, drag it into the slot, and call its `Play` method. Or change the mask material's colour, or play an `AudioSource`.

**Check:** open your mouth and hold it: one burst. Close and open: one more. Hand the phone to someone else and it works on their face unchanged.

**If it fails:**

| What you see | Likely cause |
|---|---|
| Fires continuously while open | `isOpen` is never set to true, or step 2 does not check it |
| Fires once and never again | `closedBelow` is lower than your closed-mouth value. Watch the live number and raise it |
| Never fires | `openAbove` is higher than your face reaches; lower it |
| Effect plays at the world origin | The particle system is in the scene, not a child of the face |

Commit.

## Finger quiz: how to start

This needs the native plugin bridge from Stage 8. Do Stage 8 tasks 1–3 first, then come back; the plugin layout is identical, with a different Vision request.

1. On the Swift side, the request type is the hand pose one: https://developer.apple.com/documentation/vision/vndetecthumanhandposerequest. It returns named joints for each finger, with positions from 0 to 1.
2. Send back the joints as JSON, as the [Stage 8 guide](../08-text-lens/GUIDE.md) does for text.
3. In C#, write a plain method that takes the joints of one finger and returns true or false for "extended". On paper first: for a raised finger, how does the tip's distance from the wrist compare with the knuckle's distance from the wrist?
4. Count extended fingers. Require the same count for about a second before accepting it, using a timer variable that resets whenever the count changes.

## Invisible instruments: how to start

Also needs the hand joints from the Finger quiz plugin.

1. **Theremin tone.** Unity can generate sound sample by sample in a method named `OnAudioFilterRead`: https://docs.unity3d.com/ScriptReference/MonoBehaviour.OnAudioFilterRead.html. Put a script with that method on a GameObject that has an Audio Source. Fill the buffer with a sine wave whose frequency is a public field. Get a fixed tone playing before involving hands.
2. Map the hand's height (0 to 1) onto a frequency range with `Mathf.Lerp`.
3. **Drums.** Velocity is this frame's position minus last frame's, divided by `Time.deltaTime`. Store last frame's position in a field. A hit is: velocity strongly downward *and* the position just crossed your line. Reuse the "fire once" pattern from task 5.

## Audio visualiser: how to start

No plugin needed.

1. Add an **Audio Source**. Start the microphone and make it that source's clip; the [Microphone](https://docs.unity3d.com/ScriptReference/Microphone.html) page has the pattern. Add a microphone usage description in **Player → iOS** settings or the app will crash on launch.
2. Loudness: read recent samples from the source, average their absolute values, and scale one cube by the result.
3. Bands: `GetSpectrumData` fills an array with the strength of each frequency: https://docs.unity3d.com/ScriptReference/AudioSource.GetSpectrumData.html. Sum ranges of that array into eight bands and set the height of eight cubes.
4. This quest uses the rear camera and planes, so build it in a copy of your Stage 2 project, not in `FaceMask`.

## Committing your work

```bash
git add -A
```

```bash
git commit -m "Stage 7: task 4, blend shape values on screen"
```

```bash
git push
```

Screenshots and recordings of faces, yours or other people's, stay out of the repo unless that person has agreed.
