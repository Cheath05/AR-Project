# Stage 11 — How to start

The [brief](README.md) says *what* to build. This guide says *how to go about it*: which files, what kind of code, in what order, and how to tell each step worked. It gives you empty skeletons. Filling them in is the stage.

## Before you write anything

This stage has two halves that you build separately:

1. **A server** on your Mac, written in Python. It stores your to-dos, timers and events and answers requests. Tasks 1–2. No Unity involved.
2. **A Unity app** that asks the server for data and draws it. Tasks 3–5.

You have not written a server or Python before. Both are smaller than they sound: the whole server is one file of about a hundred lines.

**Which server framework?** The brief offers FastAPI (Python) or Express (JavaScript). **Use FastAPI.** Three reasons: Python reads close to plain English; FastAPI builds a test page for your API automatically, so you can click buttons to try every request; and the Stage 12 assistant examples are in Python, so you carry straight on in the same file.

Spend 2–3 hours on Python first: sections 3–5 of the official tutorial (numbers, text, lists, `if`, `for`, functions, dictionaries). https://docs.python.org/3/tutorial/ You already know every idea from C#; only the spelling changes.

## New coding ideas in this stage

| Idea | Where you meet it | Learn it here |
|---|---|---|
| HTTP: requests, responses, methods (GET, POST…), status codes | Task 1 | https://developer.mozilla.org/en-US/docs/Web/HTTP/Overview |
| JSON, the text format both sides speak | Task 1 | https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Scripting/JSON |
| Python basics | Task 2 | https://docs.python.org/3/tutorial/ |
| Virtual environments (a private set of Python packages per project) | Task 2 | https://docs.python.org/3/library/venv.html |
| FastAPI: a function becomes a URL | Task 2 | https://fastapi.tiangolo.com/tutorial/first-steps/ |
| Receiving JSON in a request | Task 2 | https://fastapi.tiangolo.com/tutorial/body/ |
| `curl`, making requests from the terminal | Task 2 | https://curl.se/docs/tutorial.html |
| UnityWebRequest, making requests from Unity | Task 5 | https://docs.unity3d.com/ScriptReference/Networking.UnityWebRequest.html |
| Coroutines, waiting for a reply without freezing the app | Task 5 | https://docs.unity3d.com/Manual/Coroutines.html |
| World-space UI | Task 3 | https://docs.unity3d.com/Packages/com.unity.ugui@2.0/manual/HOWTO-UIWorldSpace.html |

## Files you will add

```
11-pocket-hud/
├── API.md                      (task 1)
├── server/
│   ├── README.md               how to start the server (write it for future you)
│   ├── main.py                 the endpoints
│   ├── store.py                loading and saving data.json
│   ├── requirements.txt        the packages the server needs
│   ├── .env.example            placeholder settings, safe to commit
│   ├── .gitignore              lists data.json so your real to-dos stay private
│   └── data.json               created by the server; not committed
└── PocketHud/                  (Unity project)
    └── Assets/Scripts/
        ├── Api/
        │   ├── Models.cs       C# classes matching the JSON
        │   └── ApiClient.cs    every request to the server lives here
        └── Hud/
            ├── LockMode.cs     an enum: HeadLocked, WorldLocked, LooseFollow
            ├── PanelFollower.cs
            ├── ClockPanel.cs
            ├── TodoPanel.cs
            ├── TimerPanel.cs
            ├── CalendarPanel.cs
            └── HudController.cs
```

`.venv/` and `.env` are already ignored by the repo's root `.gitignore`.

## Task by task

### Task 1 — Design the API

No code. Create `API.md`. For every request, write four things. An example of the layout (design your own fields):

```
## List to-dos
GET /todos
Response 200:
{ "todos": [ { "id": 1, "title": "...", "done": false } ] }

## Add a to-do
POST /todos
Request body:
{ "title": "..." }
Response 201:
{ "id": 2, "title": "...", "done": false }
```

Cover: list / add / tick off / delete a to-do; list / start / cancel a timer; list today's events / add an event.

Two design rules that save pain later:

- **Wrap lists in an object** (`{ "todos": [...] }`, not a bare `[...]`). Unity's `JsonUtility` cannot read a bare list.
- **A timer stores when it ends, not how long is left.** Store an end time (and the original duration). Every client works out "time remaining" from its own clock. Why does this answer the brief's question about every client showing the same time?

**Check:** for each panel in the brief, can you point to the request that feeds it? For each button a person could tap, is there a request for it?

Commit.

### Task 2 — Run it locally

**Step A: set up Python.** In the VS Code terminal:

```bash
python3 --version
```

If that fails, install Python with `brew install python`. Then:

```bash
cd projects/11-pocket-hud && mkdir server && cd server
```

```bash
python3 -m venv .venv
```

```bash
source .venv/bin/activate
```

Your prompt now starts with `(.venv)`. You must run that `source` line every time you open a new terminal for this server.

```bash
pip install "fastapi[standard]"
```

```bash
pip freeze > requirements.txt
```

**Step B: the smallest possible server.** Create `main.py`:

```
from fastapi import FastAPI

app = FastAPI()


@app.get("/health")
def health():
    # return a small dictionary, e.g. a status of "ok"
    ...
```

The line starting with `@` says "when a GET request arrives for `/health`, run the function below". Whatever the function returns is sent back as JSON. Replace the `...` with a `return`.

Run it:

```bash
fastapi dev main.py
```

**Check:** open http://127.0.0.1:8000/health in a browser and see your JSON. Then open http://127.0.0.1:8000/docs: FastAPI has built a page listing your endpoint with a **Try it out** button. In a second terminal:

```bash
curl http://127.0.0.1:8000/health
```

Leave `fastapi dev` running. It restarts itself each time you save.

**Step C: to-dos, kept in memory.** Add to `main.py`, one endpoint at a time, testing each before writing the next:

```
from pydantic import BaseModel


class NewTodo(BaseModel):
    # the fields a client sends when adding a to-do
    ...


todos = []        # a list of dictionaries, for now
next_id = 1


@app.get("/todos")
def list_todos():
    # return the list, wrapped in an object as API.md says
    ...


@app.post("/todos", status_code=201)
def add_todo(new_todo: NewTodo):
    # 1. build a dictionary with an id, the title, done = False
    # 2. add it to the list and increase next_id
    # 3. return it
    ...


@app.post("/todos/{todo_id}/done")
def mark_done(todo_id: int):
    # 1. find the to-do with that id
    # 2. not found? raise HTTPException(status_code=404)
    # 3. set done, return it
    ...
```

Test adding one from the terminal. `-X` sets the method, `-H` adds a header, `-d` is the body:

```bash
curl -X POST http://127.0.0.1:8000/todos -H "content-type: application/json" -d '{"title": "Test from curl"}'
```

```bash
curl http://127.0.0.1:8000/todos
```

**Step D: survive a restart.** Stop the server (`Ctrl + C`), start it again, list the to-dos: they are gone. Create `store.py` with two functions:

```
import json
from pathlib import Path

DATA_FILE = Path(__file__).parent / "data.json"


def load():
    # 1. if the file does not exist, return empty starting data
    # 2. otherwise open it and return json.load(...)
    ...


def save(data):
    # open the file for writing and json.dump(...) into it
    ...
```

In `main.py`, load once at the top and call `save` after every change. Create `server/.gitignore` containing the single line `data.json`.

**Step E: timers and events.** Same pattern. For timers, look up Python's `datetime` to get the current time and add seconds to it.

**Step F: reach it from your phone.** `fastapi dev` only answers requests from the Mac itself. To let other devices on your Wi-Fi in, stop it and run:

```bash
fastapi run main.py
```

Find your Mac's address on the network:

```bash
ipconfig getifaddr en0
```

It prints something like `192.168.1.23`. On your iPhone, on the same Wi-Fi, open Safari and go to `http://192.168.1.23:8000/docs` (with your number). `localhost` and `127.0.0.1` always mean "this device", so on the phone they would point at the phone.

Write these steps into `server/README.md` so you can restart the server in a month without rereading this guide.

**Check:** every request in `API.md` works from `curl`, data survives a restart, and the `/docs` page loads on your phone.

**If it fails:**

| What you see | Likely cause |
|---|---|
| `command not found: fastapi` | The virtual environment is not active; run the `source` line |
| `Address already in use` | Another copy of the server is still running in a different terminal |
| `422 Unprocessable Entity` | The JSON you sent does not match the model's fields, or the content-type header is missing |
| `404 Not Found` | The path or method differs from what the `@app...` line says |
| Works on the Mac, phone cannot connect | Using `fastapi dev`, not `fastapi run`; different Wi-Fi networks; or macOS asked to allow incoming connections and you clicked Deny (System Settings → Network → Firewall) |
| `IndentationError` | Python uses indentation where C# uses `{ }`; the lines inside a function must line up |

Commit. Check `git status` first: `data.json`, `.venv` and `.env` must not be listed.

### Task 3 — HUD panels

**Plain English:** build each panel as a prefab that can display data handed to it. No networking yet.

1. New Unity project `PocketHud`, set up for AR as in earlier stages.
2. For each panel, make a prefab: a Canvas set to **World Space**, scaled down to roughly 0.3 m wide, with text labels. Read the world-space UI page linked above for the scaling steps.
3. Each panel script has one public method that takes data and updates the labels:

```
public class TodoPanel : MonoBehaviour
{
    // references to the text labels / a row prefab

    public void Show(TodoList data)
    {
        // 1. clear the old rows
        // 2. make one row per to-do
        // 3. show a tick or strike-through for done ones
    }

    public void ShowOffline() { /* task 5 */ }
}
```

4. `Models.cs` holds plain classes matching your JSON exactly. Field names must be spelled the same as in the JSON:

```
[System.Serializable]
public class Todo { /* public fields: id, title, done */ }

[System.Serializable]
public class TodoList { public List<Todo> todos; }
```

5. For now, `HudController.Start()` builds fake data by hand and calls each panel's `Show`.

**Check:** on the phone, panels float in front of you with made-up content. Hold the phone at arm's length: can you read every line? If not, cut text or enlarge it.

### Task 4 — Locking modes

**Plain English:** one script, attached to every panel, moves the panel according to a mode.

`LockMode.cs` is a three-value enum. `PanelFollower.cs`:

```
public class PanelFollower : MonoBehaviour
{
    public LockMode mode;
    // settings: an offset from the camera (e.g. slightly left, 0.6 m ahead),
    //           a follow speed, an angle at which loose-follow starts moving

    private Transform cam;

    private void LateUpdate()
    {
        // HeadLocked:  every frame, position = camera position + offset in the
        //              camera's own directions; face the camera
        // WorldLocked: do nothing (the panel was placed once, ideally on an anchor)
        // LooseFollow: 1. where WOULD head-locked put me?
        //              2. how far, in angle, am I from the centre of view?
        //              3. only if beyond the threshold, move a fraction of the
        //                 way there this frame
    }

    public void CycleMode() { /* next mode; wire to a tap on the panel */ }
}
```

Useful things to look up: `Transform.TransformPoint`, `Vector3.Lerp` (https://docs.unity3d.com/ScriptReference/Vector3.Lerp.html), `Vector3.Angle`, `Quaternion.LookRotation`.

Build head-locked first, then world-locked, then loose-follow.

**Check:** tap a panel to cycle its mode and feel the difference. Write which mode suits which panel in `docs/log.md`, as the brief asks.

### Task 5 — Live data

**Plain English:** `ApiClient` is the only script that talks to the server. Panels ask it for data.

A web request takes time, so it runs in a coroutine. The general shape of a GET, from the Unity docs:

```
private IEnumerator GetJson(string path, System.Action<string> onSuccess, System.Action onError)
{
    using (UnityWebRequest request = UnityWebRequest.Get(baseUrl + path))
    {
        yield return request.SendWebRequest();
        // 1. check request.result: did it succeed?
        // 2. on success, pass request.downloadHandler.text to onSuccess
        // 3. otherwise call onError
    }
}
```

`ApiClient.cs` skeleton:

```
public class ApiClient : MonoBehaviour
{
    [SerializeField] private string baseUrl;   // set in the Inspector, e.g. http://192.168.1.23:8000

    public void GetTodos(System.Action<TodoList> onDone, System.Action onError) { }
    public void MarkDone(int id, System.Action onDone, System.Action onError)  { }
    public void StartTimer(int seconds, System.Action onDone, System.Action onError) { }
    // ...one method per request in API.md
}
```

Inside `GetTodos`, the JSON text becomes a C# object with `JsonUtility.FromJson<TodoList>(text)`. For POST requests with a JSON body, read the `UnityWebRequest.Post` page for the version that takes a content type: https://docs.unity3d.com/ScriptReference/Networking.UnityWebRequest.Post.html

Then, in `HudController`:

1. Replace the fake data with `ApiClient` calls.
2. Refresh on a timer. A coroutine with `yield return new WaitForSeconds(5)` in a loop does it. The clock needs no server at all; a running timer needs the server only once, then counts down locally from the end time.
3. On error, call each panel's `ShowOffline()` and keep trying.

**Two iPhone hurdles, both expected:**

- **iOS blocks plain `http://` by default.** Unity has a Player setting that allows HTTP connections; search Project Settings → Player for "HTTP" and check the manual for its exact name in your version. Allow it for development builds.
- **iOS asks permission to find devices on your local network** the first time. Tap Allow. If you tapped Don't Allow, turn it back on under Settings → Privacy & Security → Local Network.

**Check, in this order:**

1. `curl` still works from the Mac.
2. Safari on the phone still loads `/docs`.
3. The HUD shows your real to-dos.
4. Add a to-do with `curl`; within a few seconds it appears in AR.
5. Tick one off in AR; `curl` shows it done.
6. Stop the server; the HUD says offline. Start it; the HUD recovers.

**If it fails:**

| What you see | Likely cause |
|---|---|
| Safari on the phone works, Unity app does not | HTTP not allowed in Player settings, or local network permission denied |
| `Cannot connect to destination host` | Wrong address in `baseUrl`, or the Mac's address changed (run `ipconfig getifaddr en0` again) |
| Request succeeds but the object's fields are empty | C# field names do not match the JSON names exactly, or the class lacks `[System.Serializable]` |
| `FromJson` returns nothing for a list | The server returned a bare list; wrap it in an object |
| App freezes during requests | The request is not inside a coroutine |

Do not commit your home network address. Leave `baseUrl` blank in the committed scene, or load it from a file in a `private/` folder.

Commit.

## Lock-screen twin: how to start

1. Read the "How these fit with Unity" section of [docs/apple-services.md](../../docs/apple-services.md) first. A widget is a separate Swift target added to the Xcode project Unity generates, so build from Unity with **Append** to keep it between builds.
2. In Xcode: **File → New → Target → Widget Extension**. Tick the Live Activity box. Run the template unchanged and get it onto your home screen before altering it.
3. Simplest data route: the widget asks your server directly with Swift's `URLSession`. Start with a to-do count.
4. For a countdown, SwiftUI's `Text` can display a timer that ticks by itself when given an end date. This is why task 1 stored the end time.
5. The Live Activity starts from the app, so it needs one native plugin function, called from Unity when a timer starts.

## Committing your work

```bash
git status
```

```bash
git add -A
```

```bash
git commit -m "Stage 11: task 2, to-do endpoints working"
```

```bash
git push
```

Read `git status` before every `git add` in this stage. `data.json`, `.env` and `.venv/` must never appear in it.
