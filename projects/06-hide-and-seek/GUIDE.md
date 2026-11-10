# Stage 6 — How to start

The [brief](README.md) says *what* to build. This guide says *how to go about it*: which files, which Inspector settings, in what order, and how to tell each step worked. It does not contain the finished code.

Most of this stage is switching on components and looking carefully at what they do. There is less code than in Stage 5, and more observing.

## Before you write anything

1. **Find out whether your iPhone has LiDAR.** Pro models from the iPhone 12 Pro on do; look for the small black circle beside the rear cameras, or check Apple's tech specs for your model. Write the answer in your log. It decides which path you take below.
2. In Finder, duplicate your Stage 2 Unity project folder into this folder and rename it `HideAndSeek`. Open it from Unity Hub with **Add → Add project from disk**. Change the bundle identifier (**Project Settings → Player → iOS**) so it installs as a separate app.
3. Build it to your phone unchanged. Tap-to-place should still work.
4. Pick a character: any model with some height, so there is something to hide. A capsule is fine to begin.

### What your phone can do

From the ARKit plug-in manual ([occlusion](https://docs.unity3d.com/Packages/com.unity.xr.arkit@6.0/manual/arkit-occlusion.html), [meshing](https://docs.unity3d.com/Packages/com.unity.xr.arkit@6.0/manual/arkit-meshing.html)):

| Feature | With LiDAR | Without LiDAR |
|---|---|---|
| Environment depth (furniture hides things) | Yes | No |
| People occlusion (human stencil and depth) | Yes | Yes, on recent iPhones |
| Room mesh | Yes | No |
| Light estimation | Yes | Yes |

Without LiDAR you do task 1 and 3 with the *human* depth images, skip the environment part of task 2, and use the planes fallback in task 4. You still learn every concept.

## New coding ideas in this stage

| Idea | Where you meet it | Learn it here |
|---|---|---|
| Events: `+=` to listen, `-=` to stop, in `OnEnable` / `OnDisable` | Tasks 4 and 5 | https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/events/ |
| Nullable values: a `float?` that might hold nothing | Task 5 | https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/nullable-value-types |
| Textures, and showing one on screen for debugging | Task 1 | https://docs.unity3d.com/Packages/com.unity.ugui@2.0/manual/UICanvas.html |
| What a shader and a material are (concept only) | Task 4, Portal door | https://docs.unity3d.com/Manual/shader-introduction.html |
| Layers: grouping objects so code or cameras can treat them differently | Task 4 | https://docs.unity3d.com/Manual/Layers.html |
| The `Light` component from script | Task 5 | https://docs.unity3d.com/ScriptReference/Light.html |
| Collision detection modes on a Rigidbody | Marble run | https://docs.unity3d.com/Manual/ContinuousCollisionDetection.html |

## Files you will add

```
06-hide-and-seek/
└── HideAndSeek/                      (copy of your Stage 2 project)
    └── Assets/
        ├── Materials/
        │   └── MeshWireframe.mat     (task 4)
        ├── Prefabs/
        │   └── RoomMeshChunk.prefab  (task 4, LiDAR only)
        └── Scripts/
            ├── FpsCounter.cs         (do this first)
            ├── DepthDebugView.cs     (task 1)
            ├── OcclusionControls.cs  (tasks 2 and 3)
            ├── RoomMeshToggle.cs     (task 4)
            └── LightMatcher.cs       (task 5)
```

## Where the components go

Select **XR Origin → Camera Offset → Main Camera** in the Hierarchy. This camera already has **AR Camera Manager** and **AR Camera Background**. The occlusion manager must sit beside them, on that same camera.

## Do this first: a frame-rate counter

You are asked to judge cost, so you need a number on screen.

`FpsCounter.cs`, on any GameObject, with a TextMeshPro text on a Canvas dragged into its slot:

```csharp
using UnityEngine;
using TMPro;

public class FpsCounter : MonoBehaviour
{
    [SerializeField] private TMP_Text label;

    void Update()
    {
        // 1. Time.unscaledDeltaTime is how long the last frame took, in seconds
        // 2. Frames per second is 1 divided by that
        // 3. Show it in the label (updating every frame flickers; try every half second)
    }
}
```

**Check:** a number near 60 on the phone. Write it down as your baseline.

## Task by task

### Task 1 — Turn depth on and look at it

**Plain English:** switch on the occlusion manager, then put its depth picture on screen so you can see what the phone thinks is near and far.

**Editor steps:**

1. Select the Main Camera → **Add Component → AR Occlusion Manager**.
2. In its Inspector, set **Environment Depth Mode** to **Fastest** (LiDAR), or **Human Segmentation Depth Mode** to **Fastest** (no LiDAR).
3. On your Canvas add **UI → Raw Image**, about a quarter of the screen, in a corner. A Raw Image is a rectangle that displays a texture.

`DepthDebugView.cs`, on the Raw Image:

```csharp
using UnityEngine;
using UnityEngine.UI;
using UnityEngine.XR.ARFoundation;

public class DepthDebugView : MonoBehaviour
{
    [SerializeField] private AROcclusionManager occlusionManager;
    [SerializeField] private RawImage image;

    void Update()
    {
        // 1. Ask the occlusion manager for its depth texture
        //    (environmentDepthTexture with LiDAR, humanDepthTexture without)
        // 2. If it is null, there is no depth yet: return
        // 3. Give it to image.texture
    }
}
```

The texture stores distance in metres in one colour channel, so it will look mostly red or very dark, and may appear rotated. That is expected. You are checking that the shapes of your room are in it.

**Check:** hold your hand close, then far. The picture changes. Now look at the edge of a table, and at a wall 5 metres away, and note what you see for the brief's questions.

**If it fails:**

| What you see | Likely cause |
|---|---|
| Raw Image stays white | The texture is null: the mode is Disabled, or this phone does not support that depth type |
| Nothing with Environment Depth | No LiDAR. Use the human depth texture |
| `NullReferenceException` | A slot was not filled in the Inspector |
| Picture is sideways | Expected. The sensor is landscape. Rotate the Raw Image to compare |

Commit.

### Task 2 — Environment occlusion (LiDAR)

**Plain English:** with environment depth on, real things nearer than the character should now cover it. Then compare the quality settings.

There may be nothing to code for the hiding itself: AR Foundation's camera background uses the depth image once the manager is on. Place the character and walk so a chair is between you and it.

To compare settings without rebuilding each time, add three UI buttons and `OcclusionControls.cs` on the camera:

```csharp
using UnityEngine;
using UnityEngine.XR.ARFoundation;
using UnityEngine.XR.ARSubsystems;

public class OcclusionControls : MonoBehaviour
{
    [SerializeField] private AROcclusionManager occlusionManager;

    public void SetEnvironmentQuality(int level)
    {
        // 1. Turn level (0, 1, 2, 3) into an EnvironmentDepthMode value:
        //    Disabled, Fastest, Medium, Best
        // 2. Assign it to occlusionManager.requestedEnvironmentDepthMode
    }

    public void ToggleSmoothing()
    {
        // flip occlusionManager.environmentDepthTemporalSmoothingRequested
    }
}
```

Wire each button's **On Click** list to `SetEnvironmentQuality` with a different number.

**Check:** the chair hides the character. For each setting, write down the frame rate and how clean the edge looks.

**If it fails:**

| What you see | Likely cause |
|---|---|
| Character always in front | Mode is Disabled; or **Occlusion Preference Mode** is set to prefer humans or to No Occlusion |
| Works but the edge wobbles | Normal. Try Temporal Smoothing and a higher mode |
| Character vanishes entirely | It is placed below the floor or inside furniture |
| Nothing changes on a project using a custom render setup | The AR background feature from your Stage 2 setup is missing on the active renderer. Compare with the samples project |

**No LiDAR:** skip to task 3, then use the planes fallback in task 4 to get furniture-like hiding.

Commit.

### Task 3 — People occlusion

**Plain English:** a person in front of the character hides it.

**Editor steps:** on the AR Occlusion Manager set **Human Segmentation Stencil Mode** and **Human Segmentation Depth Mode** to **Fastest**. The *stencil* says which pixels are a person; the *depth* says how far away that person is. You need both.

Then look at **Occlusion Preference Mode**. The manual says environment and human depth are never both active, so this setting picks one. Add a method to `OcclusionControls.cs` to switch it at runtime:

```csharp
    public void PreferPeople(bool people)
    {
        // set occlusionManager.requestedOcclusionPreferenceMode to the
        // "prefer human" or "prefer environment" value of OcclusionPreferenceMode
    }
```

Exact option names are on the manual page: https://docs.unity3d.com/Packages/com.unity.xr.arfoundation@6.0/manual/features/occlusion/occlusion-manager.html

**Check:** hold your hand between the phone and the character: your hand covers it. Then find where it fails. Try a hand very close to the lens, a person side-on, a person far away.

**If it fails:**

| What you see | Likely cause |
|---|---|
| Hand never hides it | Stencil on but depth off, or the reverse |
| People hide it but furniture stopped | Preference mode is on humans. That is the trade-off; note it |
| Fringe of background around the person | Normal for segmentation. Try Best and compare cost |

Commit.

### Task 4 — Meshing, with a fallback

#### With LiDAR

**Plain English:** ask ARKit for a 3D model of the room, show it as lines, then hide the lines but keep it solid for physics.

**Editor steps:**

1. Make the chunk prefab: an empty GameObject with **Mesh Filter**, **Mesh Renderer** and **Mesh Collider**. Give the renderer a bright material. For a wireframe look, the AR Foundation samples include a mesh material you can study. Drag it into `Assets/Prefabs/` as `RoomMeshChunk` and delete it from the scene.
2. Right-click **XR Origin → Create Empty**, name it `Meshing`. It must be a *child* of the XR Origin. Add **AR Mesh Manager** to it.
3. Drag `RoomMeshChunk` into its **Mesh Prefab** slot.
4. Leave **Density** alone: the ARKit manual says ARKit does not implement it. **Normals** is supported.

Build and walk around slowly. The room should fill in with mesh.

`RoomMeshToggle.cs`, on the `Meshing` object, wired to a UI button:

```csharp
using UnityEngine;
using UnityEngine.XR.ARFoundation;

public class RoomMeshToggle : MonoBehaviour
{
    [SerializeField] private ARMeshManager meshManager;
    private bool visible = true;

    void OnEnable()
    {
        // listen to meshManager.meshesChanged so new chunks get the current visibility
    }

    void OnDisable()
    {
        // stop listening
    }

    public void ToggleVisible()
    {
        // 1. Flip visible
        // 2. Loop over meshManager.meshes (a list of MeshFilter)
        // 3. For each, get its MeshRenderer and set enabled to visible
    }
}
```

Turning the renderer off leaves the collider on, so the mesh is invisible but solid. Hiding is already handled by the depth image from task 2.

To prove collision: give a sphere a **Rigidbody**, spawn it above a table from a button, and watch it land.

#### Without LiDAR

There is no mesh. Use the planes you already detect.

1. Open your AR plane prefab. Check it has a **Mesh Collider**. If so, a Rigidbody sphere already lands on detected tables and floors. Test that first.
2. For hiding: a plane can hide things *below or behind* it if its material draws nothing visible but still blocks what is behind. Research "depth mask shader" and Unity's `ColorMask` setting. Make a second material with such a shader and put it on the plane prefab.
3. Write in your log what planes cannot represent (a chair leg, a curved sofa, anything not flat).

**Check:** something virtual lands on a real surface and stays there.

**If it fails:**

| What you see | Likely cause |
|---|---|
| No mesh ever appears | No LiDAR; or the AR Mesh Manager is not a child of the XR Origin; or no Mesh Prefab assigned |
| Mesh appears far from the room | The `Meshing` object is not at position 0, rotation 0, scale 1 |
| Sphere falls through | No Mesh Collider on the prefab, or the sphere is fast and tiny (see Marble run) |
| Frame rate collapses | Lots of mesh with colliders updating. Note the number for the brief |

Commit.

### Task 5 — Light estimation

**Plain English:** every frame, ARKit reports how bright the room is. Copy that onto your scene's light.

**Editor steps:**

1. On the Main Camera's **AR Camera Manager**, open **Light Estimation** and tick **Ambient Intensity** and **Ambient Color**.
2. Make sure the scene has one **Directional Light**, and that your character's material reacts to light (a default Lit material does; an Unlit one will never change).

`LightMatcher.cs`, on the Directional Light:

```csharp
using UnityEngine;
using UnityEngine.XR.ARFoundation;

public class LightMatcher : MonoBehaviour
{
    [SerializeField] private ARCameraManager cameraManager;
    private Light sceneLight;

    void Awake()
    {
        // get the Light component on this GameObject
    }

    void OnEnable()
    {
        // listen to cameraManager.frameReceived
    }

    void OnDisable()
    {
        // stop listening
    }

    void OnFrame(ARCameraFrameEventArgs args)
    {
        // 1. args.lightEstimation.averageBrightness is a float? (it may hold nothing)
        // 2. If it has a value, use it to set sceneLight.intensity
        // 3. Do the same for colour temperature if it has a value
        // 4. While testing, show the raw numbers on screen
    }
}
```

A `float?` has `.HasValue` and `.Value`. Always check the first before reading the second.

Which values to expect: the [ARKit camera page](https://docs.unity3d.com/Packages/com.unity.xr.arkit@6.0/manual/arkit-camera.html) says world tracking provides ambient intensity and colour temperature only. Main light direction and spherical harmonics come with face tracking. So ticking the other boxes here gives you nothing; confirm that by printing them.

**Check:** switch the room light off. The number on screen drops and the character darkens.

**If it fails:**

| What you see | Likely cause |
|---|---|
| Values never have a value | No light estimation options ticked on the AR Camera Manager |
| Number changes, character does not | Unlit material, or another light is doing the lighting |
| Character goes pitch black | The brightness scale needs adjusting. Print it in a bright and a dark room and choose a multiplier |

Record the final frame rate with everything on. Commit.

## Life-size models: how to start

1. Import the model. Add a 1 × 1 × 1 cube next to it in the editor: that cube is one metre. Scale the model until it is the right number of cubes tall, then delete the cube.
2. Make it the object your Stage 2 placement code spawns.
3. Labels: a world-space TextMeshPro as a child of each part. A small script on each label turns it toward the camera every frame; see `Transform.LookAt`: https://docs.unity3d.com/ScriptReference/Transform.LookAt.html
4. For "hide the label when its part is hidden", start simple: hide labels whose part faces away from the camera. Compare the direction to the camera with the part's facing direction.

## Portal door: how to start

Build in three steps, each one working before the next.

1. **A door on the wall.** A flat quad placed with your Stage 2 code on a vertical plane (enable vertical detection on the plane manager).
2. **The other world, visible only through it.** Read the stencil page: https://docs.unity3d.com/Manual/SL-Stencil.html. The door's material writes a number into the stencil buffer and draws no colour. Every material in the other world draws only where that number is. In the Universal Render Pipeline you can set stencil rules per layer without writing shader code, using a Render Objects feature: https://docs.unity3d.com/Manual/urp/renderer-features/renderer-feature-render-objects.html. Put the other world on its own layer.
3. **Walking through.** A small script that each frame works out which side of the door plane the camera is on (look up `Vector3.Dot`). When the sign changes, flip the rule so the other world draws everywhere *except* through the door.

## Marble run: how to start

1. Reuse the sphere spawner from task 4. Make the sphere small (2–3 cm).
2. When marbles fall through the mesh, read the collision detection page in the table above and change the Rigidbody's **Collision Detection** setting.
3. Destroy marbles that fall below the floor, or you will have thousands.

## Room scanner: how to start

1. Add a button that loops over `meshManager.meshes`. Each `MeshFilter` has a `mesh` with `vertices` and `triangles` arrays.
2. The OBJ format is plain text, one vertex or face per line: https://en.wikipedia.org/wiki/Wavefront_.obj_file. Build that text and write it to `Application.persistentDataPath`, as in Stage 5.
3. Each chunk's vertices are relative to that chunk. Convert them to world positions first with the chunk transform's `TransformPoint`.
4. To get the file off the phone, research enabling file sharing for your app in the Xcode project's Info settings, then use the Files app or Finder.

## Committing your work

```bash
git add -A
```

```bash
git commit -m "Stage 6: task 2, environment occlusion with quality buttons"
```

```bash
git push
```

A room scan is a 3D picture of where you live. Keep exported scans in a `private/` folder.
