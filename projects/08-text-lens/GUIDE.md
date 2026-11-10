# Stage 8 — How to start

The [brief](README.md) says *what* to build. This guide says *how to go about it*: which files, in which language, in what order, and how to tell each step worked. It does not contain the finished code.

This is the first stage with three languages in one project. That is the hard part, and it is all in tasks 1–3. Go slowly there.

## Before you write anything

1. Read "A Swift Tour" up to and including "Functions and Closures" (about an hour). Swift will look familiar after C#: https://docs.swift.org/swift-book/documentation/the-swift-programming-language/guidedtour/
2. Read Unity's four short plug-in pages in this order: [create](https://docs.unity.com/en-us/engine/6000.6/manual/platform-specific/iphone/ios-developing/plugins-for-ios/ios-native-plugin-create), [call](https://docs.unity.com/en-us/engine/6000.6/manual/platform-specific/iphone/ios-developing/plugins-for-ios/ios-native-plugin-call), [call back](https://docs.unity.com/en-us/engine/6000.6/manual/platform-specific/iphone/ios-developing/plugins-for-ios/ios-native-plugin-call-back), [Inspector settings](https://docs.unity.com/en-us/engine/6000.6/manual/platform-specific/iphone/ios-developing/plugins-for-ios/ios-native-plugin-automated-integration).
3. Copy your Stage 2 project into this folder as `TextLens` (you need planes and raycasts in task 4). Change the bundle identifier. Build it unchanged to check it still runs.
4. Know how to see the **Xcode console**: when you run from Xcode with the phone plugged in, the panel at the bottom shows `print` output from Swift and `Debug.Log` from C#. From now on this is your main debugging tool; native code cannot run in the Unity editor.

## New coding ideas in this stage

| Idea | Where you meet it | Learn it here |
|---|---|---|
| Swift basics: `func`, `let`, `var`, optionals, closures | The whole Swift file | https://docs.swift.org/swift-book/documentation/the-swift-programming-language/guidedtour/ |
| `static extern` and `[DllImport]`: declaring a function that lives in another language | Task 1 | https://learn.microsoft.com/en-us/dotnet/standard/native-interop/pinvoke |
| `#if UNITY_IOS && !UNITY_EDITOR`: code that only exists on the phone | Task 1 | https://docs.unity3d.com/Manual/platform-dependent-compilation.html |
| CPU images, and `Dispose` | Task 2 | https://docs.unity3d.com/Packages/com.unity.xr.arfoundation@6.0/manual/features/camera/image-capture.html |
| Work that finishes later, off the main thread | Tasks 2 and 3 | "Closures" in the Swift tour; Apple's Vision text article below |
| JSON as the message format between languages | Task 3 | https://docs.unity3d.com/Manual/JSONSerialization.html |
| Normalised coordinates (0 to 1) versus pixels | Task 4 | https://docs.unity3d.com/ScriptReference/Camera.ViewportToScreenPoint.html |
| `Dictionary` for "have I seen this before?" | Task 5 | https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.dictionary-2 |

## How a native plugin is laid out

Read this section twice. Everything else follows from it.

C# cannot call Swift directly. Both can speak plain **C**, so C-style functions sit in the middle. A call goes out through one, and the answer comes back through another route.

```
   C# (Unity)                     C-style functions                 Swift
 ┌────────────────┐   call   ┌─────────────────────────┐  call  ┌────────────────┐
 │ NativeBridge.cs│ ───────► │ TextLensBridge.mm       │ ─────► │ TextLens.swift │
 │ [DllImport]    │          │ extern "C" { ... }      │        │ Vision code    │
 └────────────────┘          └─────────────────────────┘        └───────┬────────┘
 ┌────────────────┐  string  ┌─────────────────────────┐  result        │
 │ TextResult     │ ◄─────── │ UnitySendMessage(...)   │ ◄──────────────┘
 │ Receiver.cs    │          │ (called from the .mm)   │
 └────────────────┘          └─────────────────────────┘
```

### The rules, from Unity's manual

**Where files go.** Source files with the extensions `.m`, `.mm`, `.c`, `.cpp`, `.h` or `.swift` placed in `Assets/Plugins/iOS` are automatically enabled for iOS. At build time Unity *copies* them into the Xcode project. Edits made inside Xcode are overwritten on the next build, so always edit the copy in `Assets/Plugins/iOS`.

**The C# side.** For each native function, one declaration with no body:

```csharp
[DllImport("__Internal")]
private static extern ReturnType FunctionName(ParameterType name);
```

`"__Internal"` means "this function is built into the app itself". The name must match the native function's name letter for letter.

**The native side, Objective-C++ (`.mm`).** Functions must be declared with C linkage so their names are not altered:

```objc
extern "C" {
    ReturnType FunctionName(ParameterType name);
}
```

**The native side, Swift.** A Swift function can be exported directly under a C name with an attribute:

```swift
@_cdecl("FunctionName")
public func anySwiftName(_ name: ParameterType) -> ReturnType {
    // body
}
```

**Getting an answer back.** Native code calls:

```objc
UnitySendMessage("GameObjectName", "MethodName", "one string");
```

Unity then finds the GameObject with that name and calls that method on it. The manual's restrictions:

- The C# method must be `public`, return `void` and take exactly one `string`.
- The call is asynchronous and arrives one frame later.
- Two GameObjects with the same name cause conflicts, so give the receiver a unique name.

Because you only get one string, you pack everything into JSON.

**Only on the device.** Native functions do not exist in the editor. Wrap every call:

```csharp
#if UNITY_IOS && !UNITY_EDITOR
    // call the native function
#else
    // return a placeholder so the editor does not error
#endif
```

**System frameworks.** Select the plugin file in the Project window; its Inspector has a setting for the operating system frameworks it depends on. Add **Vision** there if Xcode reports it missing at link time.

### Which layout to use here

- **Task 1** uses the simplest form: one Swift file with `@_cdecl`, no `.mm`.
- **From task 3** you add a small `.mm` file, because `UnitySendMessage` is a C function in Unity's own code that Swift cannot see by default. The `.mm` file can see both sides: it calls into your Swift class, and calls `UnitySendMessage` with the result. To call Swift from a `.mm` file you import a header that Xcode generates for the Swift code in the UnityFramework target; open the Xcode project and check the manual and Unity's example project for its exact name: https://github.com/Unity-Technologies/nativePlugin-example

There is a second documented way back, using a C# *delegate* passed to native code as a function pointer (see the "call back" page). It avoids the `.mm` file but needs more C# machinery. Start with `UnitySendMessage`.

## Files you will add

```
08-text-lens/
└── TextLens/
    └── Assets/
        ├── Plugins/
        │   └── iOS/
        │       ├── TextLens.swift          (tasks 1–3: all Swift code)
        │       └── TextLensBridge.mm       (task 3: C functions, UnitySendMessage)
        ├── Prefabs/
        │   └── TextLabel.prefab            (task 5)
        └── Scripts/
            ├── NativeBridge.cs             (task 1: every DllImport in one place)
            ├── BridgeTest.cs               (task 1: temporary)
            ├── FrameSender.cs              (task 2)
            ├── TextResult.cs               (task 3: plain data classes)
            ├── TextResultReceiver.cs       (task 3)
            ├── DebugOverlay.cs             (task 3)
            ├── ImageToScreen.cs            (task 4)
            └── LabelPlacer.cs              (tasks 4 and 5)
```

Create the `Plugins` and `iOS` folders by hand; the names must be exact. Unity has no "new Swift file" menu, so create `TextLens.swift` in VS Code inside that folder.

## Task by task

### Task 1 — Build the bridge

**Plain English:** C# sends the number 41 to Swift. Swift adds one and returns it. C# prints 42. No Vision, no camera.

**`TextLens.swift`:**

```swift
import Foundation

@_cdecl("TextLens_AddOne")
public func textLensAddOne(_ value: Int32) -> Int32 {
    // 1. print something so you can see in the Xcode console that Swift ran
    // 2. return the value plus one
}
```

`Int32` is Swift's name for C#'s `int`. Prefixing every exported name (`TextLens_`) avoids clashing with a function somewhere else in the app.

**`NativeBridge.cs`** is a plain static class, the only file that knows native code exists:

```csharp
using System.Runtime.InteropServices;

public static class NativeBridge
{
#if UNITY_IOS && !UNITY_EDITOR
    [DllImport("__Internal")]
    private static extern int TextLens_AddOne(int value);
#endif

    public static int AddOne(int value)
    {
#if UNITY_IOS && !UNITY_EDITOR
        // call the native function and return its result
#else
        // return a placeholder so the editor still runs
#endif
    }
}
```

**`BridgeTest.cs`**, a `MonoBehaviour` on any GameObject: in `Start`, call `NativeBridge.AddOne(41)` and `Debug.Log` the answer.

Build to Xcode and run with the console open.

**Check:** the Xcode console shows your Swift `print` line, then Unity's log line with `42`.

**If it fails:**

| What you see | Likely cause |
|---|---|
| Xcode: `Undefined symbols ... _TextLens_AddOne` | Names do not match exactly; or the Swift file is not inside `Assets/Plugins/iOS`; or `@_cdecl` is missing |
| `EntryPointNotFoundException` in the Unity editor | A native call is not wrapped in the `#if` |
| Xcode errors about Swift settings or version | Open the generated project's build settings for the UnityFramework target and check a Swift language version is set. Search the exact error text |
| It builds, but no Swift `print` | `Start` never ran: is `BridgeTest` on an active GameObject in the built scene? |
| You fixed the Swift file in Xcode and the fix vanished | Xcode's copy is overwritten on each build. Edit the one in `Assets/Plugins/iOS` |

This task is expected to take a whole session. Commit when 42 appears.

### Task 2 — Send a camera frame to Vision

**Plain English:** a few times a second, take the camera picture AR Foundation already has, hand its pixels to Swift, and have Vision print any text it finds.

**The C# half, `FrameSender.cs`**, on the Main Camera:

```csharp
using Unity.Collections;
using UnityEngine;
using UnityEngine.XR.ARFoundation;
using UnityEngine.XR.ARSubsystems;

public class FrameSender : MonoBehaviour
{
    [SerializeField] private ARCameraManager cameraManager;
    [SerializeField] private float secondsBetweenFrames = 0.5f;

    private float timer;
    private bool waitingForResult;

    void Update()
    {
        // 1. Count the timer up. If it is not time yet, or we are still
        //    waiting for the last answer, return
        // 2. Reset the timer and call SendFrame
    }

    void SendFrame()
    {
        // 1. cameraManager.TryAcquireLatestCpuImage(out XRCpuImage image)
        //    If it returns false, return
        // 2. Fill in an XRCpuImage.ConversionParams:
        //    inputRect (the whole image), outputDimensions (make it smaller),
        //    outputFormat, transformation
        // 3. Make a buffer of the right size and convert the image into it
        // 4. Dispose the image. Always, even if something went wrong
        // 5. Pass the buffer, its width and its height to NativeBridge
    }
}
```

Notes for each step, all from the [image capture page](https://docs.unity3d.com/Packages/com.unity.xr.arfoundation@6.0/manual/features/camera/image-capture.html), whose sample you should read beside this:

- **Dispose** is not optional. The manual warns that holding images can stop new ones arriving. Its sample uses `try` / `finally`.
- **Size.** Halve or quarter the dimensions. Text recognition does not need full resolution, and smaller is faster.
- **Format.** The manual notes single-channel formats convert much faster than colour. Either works for Vision; pick one and tell Swift which.
- **Convert and buffer size.** `XRCpuImage` has methods for both; the exact overloads are on its API page: https://docs.unity3d.com/Packages/com.unity.xr.arfoundation@6.0/api/UnityEngine.XR.ARSubsystems.XRCpuImage.html
- **Passing pixels.** Declare the native function to take a `byte[]` plus two `int`s. The pointer Swift receives is only valid *during* the call, so Swift must copy the bytes before it returns.

Add to `NativeBridge.cs`, following the task 1 pattern: a `DllImport` line for `TextLens_Recognize(byte[] pixels, int width, int height)` and a public wrapper.

**The Swift half.** Read Apple's article first: https://developer.apple.com/documentation/vision/recognizing-text-in-images. Add to `TextLens.swift`:

```swift
import Vision
import CoreGraphics

@_cdecl("TextLens_Recognize")
public func textLensRecognize(_ pixels: UnsafePointer<UInt8>, _ width: Int32, _ height: Int32) {
    // 1. Copy the bytes into a Data value right away
    // 2. Build a CGImage from that data, width and height
    // 3. Create a text recognition request with a completion handler
    // 4. In the handler: for each observation, print its best candidate string
    //    and its boundingBox
    // 5. Run the request with an image request handler, on a background queue
}
```

Step 2 is the fiddly one. Search "CGImage from raw pixel data Swift"; you need a colour space and bitmap settings that match the format you chose in C#. Step 5 matters because recognition takes a noticeable fraction of a second; running it on the main thread freezes the app. The request class is documented here: https://developer.apple.com/documentation/vision/vnrecognizetextrequest

**Orientation.** The camera sensor is landscape. If you hold the phone upright, the image Vision receives is on its side and it will find little or nothing. Two ways to deal with it: tell the Vision request handler the image's orientation, or hold the phone in landscape while testing. Start with landscape to prove the pipeline, then fix portrait.

**Check:** point the phone at large printed text. The words appear in the Xcode console.

**If it fails:**

| What you see | Likely cause |
|---|---|
| `TryAcquireLatestCpuImage` always false | Called before the session started; or you are in the editor |
| App slows and then the camera freezes | Images are not being disposed |
| Crash in Swift with a memory error | Swift used the pointer after returning instead of copying first; or width × height × bytes-per-pixel does not match the buffer length |
| Vision returns nothing | Orientation; text too small; image mirrored or upside down. Try a different `transformation` value |
| Random characters | The pixel format in Swift does not match the one C# converted to |
| App stutters every half second | Recognition is on the main thread |

To see what Vision is being given, temporarily show the converted buffer in a Raw Image (the image capture page's sample does this). Commit.

### Task 3 — Get results back into Unity

**Plain English:** Swift packs the strings and boxes into one JSON string. The `.mm` file sends it to a named GameObject. C# unpacks it and shows it.

**Decide the JSON shape first** and write an example by hand:

```
{ "items": [ { "text": "...", "x": 0.0, "y": 0.0, "w": 0.0, "h": 0.0 } ] }
```

**`TextResult.cs`**: two plain `[Serializable]` classes that match that shape field for field, exactly as you did for cards in Stage 5 (one item class, one wrapper with a `List`).

**`TextLensBridge.mm`**: this file now becomes the entry point C# calls, and it forwards to Swift.

```objc
#import <Foundation/Foundation.h>
// plus the import for the generated Swift header (see "Which layout to use here")

extern "C" {
    void UnitySendMessage(const char* obj, const char* method, const char* msg);

    void TextLens_Recognize(const unsigned char* pixels, int width, int height)
    {
        // 1. Call your Swift class's recognize method, passing the pixels and size
        // 2. Give it a completion block that receives the JSON string
        // 3. Inside the block, call UnitySendMessage with:
        //    the receiver's GameObject name, its method name, and the JSON
    }
}
```

**In `TextLens.swift`**, move the Vision code out of the `@_cdecl` function into a class the `.mm` file can use, and remove `@_cdecl("TextLens_Recognize")` so the name is not defined twice:

```swift
@objc public class TextLens: NSObject {
    @objc public static let shared = TextLens()

    @objc public func recognize(pixels: UnsafePointer<UInt8>,
                                width: Int32,
                                height: Int32,
                                completion: @escaping (String) -> Void) {
        // 1–3. As in task 2
        // 4. In the handler: build an array of small dictionaries, one per observation
        // 5. Turn it into a JSON string (look up JSONSerialization or Codable)
        // 6. Call completion with that string, on the main queue
    }
}
```

`@objc` and `NSObject` make the class visible to the `.mm` file.

**`TextResultReceiver.cs`**, on a GameObject named **exactly** what the `.mm` file passes as the first argument (for example `TextLensReceiver`):

```csharp
using System;
using UnityEngine;

public class TextResultReceiver : MonoBehaviour
{
    public event Action<TextResultList> ResultsArrived;

    // Called by native code. Must be public, void, one string.
    public void OnTextResults(string json)
    {
        // 1. Debug.Log the raw json (until this works)
        // 2. Turn it into your wrapper object with JsonUtility
        // 3. Raise ResultsArrived so other scripts can react
    }
}
```

Then have `FrameSender` listen to `ResultsArrived` and set `waitingForResult` back to false there, and set it true when it sends. That is what stops frames piling up.

**`DebugOverlay.cs`**: listens to the same event and writes every recognised string into one TextMeshPro text on a Canvas.

**Check:** point at text; the words appear on the phone's screen, not only in Xcode.

**If it fails:**

| What you see | Likely cause |
|---|---|
| Xcode: Swift header file not found | The import name is wrong. Find the generated header's real name in the Xcode project |
| Xcode: duplicate symbol `_TextLens_Recognize` | The `@_cdecl` version is still in the Swift file |
| Swift prints results, C# never receives them | GameObject name or method name differs by a letter; the method is not `public`; the GameObject is inactive |
| `ArgumentException: JSON parse error` | Log the raw string. Field names or nesting do not match your classes |
| Text contains quotes and breaks the JSON | You built the JSON by gluing strings. Use a JSON encoder in Swift |
| Results stop after the first | `waitingForResult` is never cleared, often because the empty-result case does not call the completion |

Commit. The slow part of the stage is now over.

### Task 4 — Map the 2D box to a 3D position

**Plain English:** turn "the box is at these fractions of the image" into "this pixel on the screen", then ask AR Foundation what real surface is under that pixel.

**Draw this on paper first.** Three rectangles are involved and they are not the same:

- **Vision's box** is in fractions from 0 to 1 of the *image you sent*, measured from the image's bottom-left corner.
- **The camera image** is not the same shape as the screen. It is cropped to fill the display, and rotated if you hold the phone upright.
- **Unity's screen** is in pixels, measured from the screen's bottom-left corner.

**`ImageToScreen.cs`**, a plain static class:

```csharp
using UnityEngine;

public static class ImageToScreen
{
    public static Vector2 BoxCentreToScreenPoint(TextItem item)
    {
        // 1. Find the centre of the box in image fractions (x + w/2, y + h/2)
        // 2. Correct for rotation, mirroring and cropping between image and screen
        // 3. Multiply by Screen.width and Screen.height to get pixels
    }
}
```

Do step 2 last. First skip it, hold the phone in landscape, and see how far off the result is. Then work out the correction. AR Foundation provides the exact transform between camera image and display as the `displayMatrix` on the camera frame event (`ARCameraFrameEventArgs`, the same event you used for light estimation in Stage 6); that is the proper tool once the simple version is close.

**Seeing whether you are right:** put a small UI Image (a dot) on a Canvas and move it to the computed screen point each time results arrive. It should sit on the text. Get the dot right before touching 3D.

**Then 3D**, in `LabelPlacer.cs`:

```csharp
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.XR.ARFoundation;
using UnityEngine.XR.ARSubsystems;

public class LabelPlacer : MonoBehaviour
{
    [SerializeField] private ARRaycastManager raycastManager;
    [SerializeField] private TextResultReceiver receiver;
    [SerializeField] private GameObject labelPrefab;

    private readonly List<ARRaycastHit> hits = new List<ARRaycastHit>();

    void OnEnable()  { /* listen to receiver.ResultsArrived */ }
    void OnDisable() { /* stop listening */ }

    void OnResults(TextResultList results)
    {
        // For each item:
        // 1. Get its screen point from ImageToScreen
        // 2. Raycast from that point, as in Stage 2
        // 3. If nothing was hit, skip this item
        // 4. Take the pose of the first hit and hand it to PlaceOrUpdate
    }

    void PlaceOrUpdate(string text, Pose pose)
    {
        // task 5
    }
}
```

The raycast only hits detected planes, so test with a page lying on a table the app has already found.

**Check:** the debug dot sits on the text, and a log line prints a believable world position (roughly table height below the phone).

**If it fails:**

| What you see | Likely cause |
|---|---|
| Dot mirrored top-to-bottom | An origin mismatch. Try `1 - y` and work out *why* it was needed |
| Dot correct in landscape, wrong in portrait | The image is rotated relative to the screen; x and y swap |
| Dot close at the centre, drifting toward the edges | Cropping: the image and the screen have different aspect ratios |
| Raycast never hits | No plane under the text yet. Scan the table first |

Commit.

### Task 5 — Place a label in AR

**Plain English:** put a floating label at the hit position. If that text already has a label, move the old one a little; do not make another.

1. Make `TextLabel.prefab`: an empty GameObject with a world-space TextMeshPro child, scaled small (text a few centimetres high). Reuse your Stage 3 note prefab if you have one.
2. Add a field to `LabelPlacer`: `private readonly Dictionary<string, GameObject> labels = new Dictionary<string, GameObject>();`
3. Fill in `PlaceOrUpdate`:

```csharp
    void PlaceOrUpdate(string text, Pose pose)
    {
        // 1. Is there already an entry for this text in labels?
        // 2. If no: Instantiate the prefab at the pose, set its text, add it to labels
        // 3. If yes: move the existing one toward the new position
        //    (jumping straight there jitters; moving part of the way is smoother)
    }
```

Then decide the awkward cases the brief raises: the same word in two places, a word read slightly differently on the next frame, a label whose text has not been seen for a while. Write your rule in a comment before coding it.

To make the label stay put as you move, parent it to an anchor as in Stage 3.

**Check:** a label sits on the text. Walk around: it stays on the page. Keep pointing at it for ten seconds: still one label. Use your Stage 6 frame-rate counter to confirm the app stays smooth.

**If it fails:**

| What you see | Likely cause |
|---|---|
| A stack of labels | The dictionary lookup never matches. Log the keys; look for trailing spaces or different capitalisation |
| Label slides with the phone | It is a child of the camera or on a screen-space Canvas |
| Label shakes | Position is replaced outright each time. Blend toward it |
| Label lags behind when you move fast | The frame was taken a moment before the answer arrived. See question 3 in the brief |

Commit.

## Homework lens: how to start

1. **Data first.** Add to your Stage 5 data files: a `Problem` class with an `id`, a `matchText` (a phrase or number unique to that problem on the page) and a `List<string> steps`. Write three real problems as JSON by hand.
2. **Matching.** A plain static method that takes the recognised strings and your list of problems and returns the first problem whose `matchText` appears in any string. Compare in lower case. Test it in the editor with made-up strings; it needs no camera.
3. **Reveal.** A script holding the current problem and an `int` for how many steps are showing. A button adds one and rebuilds the label text from the first N steps.
4. **Into study buddy.** Copy the `Plugins/iOS` folder and the scripts across, then wrap the feature as a class implementing `IStudyMode` from Stage 5. Stop `FrameSender` in `End`.

## Vocabulary labels: how to start

1. Download a small image classification model from Apple's list: https://developer.apple.com/machine-learning/models/. Core ML overview: https://developer.apple.com/documentation/coreml
2. The Swift side is the same shape as text recognition with a different request type that wraps the model. Add a second exported function next to the first; reuse the pixel-to-image code.
3. Return the top label and its confidence as JSON, through the same `UnitySendMessage` route to a second receiver method.
4. Translation: a `Dictionary<string, string>` from English label to your target word, loaded from a JSON file you write. Show a label only when confidence is above a threshold *and* the word is in your dictionary.

## Deck scanner: how to start

1. Reuse the text lens unchanged. The card name is usually the string whose box is highest on the card: sort the results by `y` and take the top one.
2. Add a **Scan** button so a card is recorded only when you choose, not continuously.
3. Store names in a `[Serializable]` list and save it with your Stage 5 file code.

## Committing your work

```bash
git add -A
```

```bash
git commit -m "Stage 8: task 1, C# calls Swift and gets 42 back"
```

```bash
git push
```

Check that `Assets/Plugins/iOS` is included in the commit; those files are the stage. The generated Xcode project (`iOSBuild/`) is ignored and should stay that way. Photos of textbook pages and real cards go in `private/`.
