# Stage 12 — How to start

The [brief](README.md) says *what* to build. This guide says *how to go about it*: which files, what kind of code, in what order, and how to tell each step worked. It gives you empty skeletons. Filling them in is the stage.

## Before you write anything

1. Your Stage 11 server must be working. The assistant is extra endpoints on that same server.
2. Create an account at the LLM provider's console, add a small amount of credit, **set a monthly spending limit**, and create an API key. This guide uses Claude; the structure is the same for any provider.
3. Re-read the two rules in the brief. This guide assumes both.

**The rule that matters most: the API key lives only in `server/.env`.** It is never typed into a `.py` file, a C# file, a Unity Inspector field, a chat message or a commit. If you ever paste it anywhere else, revoke it in the console and make a new one.

How the pieces connect:

```
iPhone (Unity)                   Your Mac (Python server)               LLM provider
  speech → text ──POST /chat──►  add to conversation  ──── request ───►
                                 run any tools asked for ◄── reply ────
  speak reply  ◄──── JSON ─────  return final text
```

Almost all the new code is on the server. Build and test it there with `curl` before the phone is involved.

## New coding ideas in this stage

| Idea | Where you meet it | Learn it here |
|---|---|---|
| Environment variables and `.env` files (settings kept out of code) | Milestone 1 | https://pypi.org/project/python-dotenv/ |
| The Messages API: a conversation as a list of messages | Milestone 1 | https://platform.claude.com/docs/en/api/messages |
| System prompt (standing instructions for the assistant) | Milestone 1 | Same page, the `system` field |
| Tool use: the model asks your code to do something | Milestone 2 | https://platform.claude.com/docs/en/agents-and-tools/tool-use/overview |
| JSON Schema (describing what inputs a tool takes) | Milestone 2 | The `input_schema` examples on the tool-use page |
| Sending an image to the model | Milestone 3 | https://platform.claude.com/docs/en/build-with-claude/vision |
| Grabbing a camera frame in AR Foundation | Milestone 3 | https://docs.unity3d.com/Packages/com.unity.xr.arfoundation@6.0/manual/features/camera/image-capture.html |
| Speech to text on iPhone | Milestone 1 | https://developer.apple.com/documentation/speech/sfspeechrecognizer |
| Text to speech on iPhone | Milestone 1 | https://developer.apple.com/documentation/avfaudio/avspeechsynthesizer |
| Streaming replies (text arriving piece by piece) | Milestone 5 | https://platform.claude.com/docs/en/build-with-claude/streaming |

## Files you will add

Copy `projects/11-pocket-hud/server/` to `projects/12-assistant/server/` and carry on in the copy, so Stage 11 stays as a working reference. (Do not copy `.venv`; make a fresh one.)

```
12-assistant/
├── server/
│   ├── README.md
│   ├── main.py            Stage 11 endpoints + POST /chat
│   ├── store.py           from Stage 11
│   ├── assistant.py       the conversation loop
│   ├── tools.py           tool descriptions + the functions that run them
│   ├── prompts.py         the system prompt text
│   ├── memory.py          (milestone 4)
│   ├── requirements.txt
│   ├── .env               your real key. Ignored by Git. Never commit.
│   ├── .env.example       the same file with a placeholder. Commit this.
│   └── .gitignore         data.json, memory.json, conversations/
└── Assistant/             (Unity project; start from a copy of PocketHud)
    └── Assets/
        ├── Plugins/iOS/
        │   └── SpeechBridge.swift     (plus whatever glue file your Stage 8 plugin needed)
        └── Scripts/Assistant/
            ├── SpeechBridge.cs        C# side of the plugin
            ├── AssistantClient.cs     POSTs to /chat
            ├── AssistantState.cs      enum: Idle, Listening, Thinking, Speaking
            ├── PushToTalk.cs          the button
            ├── AssistantStateView.cs  shows the current state
            └── OrbView.cs             (play quest)
```

## Server setup

```bash
cd projects/12-assistant/server
```

```bash
python3 -m venv .venv && source .venv/bin/activate
```

```bash
pip install -r requirements.txt
```

```bash
pip install anthropic python-dotenv && pip freeze > requirements.txt
```

Create `.env.example` (committed):

```
ANTHROPIC_API_KEY=put-your-key-here
```

Create `.env` (not committed) with the same line and your real key. Then prove Git cannot see it:

```bash
git status
```

`.env` must not appear in the output. If it does, stop and fix `.gitignore` before going further.

## Milestone by milestone

### Milestone 1 — Push-to-talk Q&A

Do tasks 1–3 entirely on the Mac.

**Task 1: one question, one answer.**

What a request to the Messages API contains (confirmed against the docs linked above; re-check there, because model names change):

| Field | What it is |
|---|---|
| `model` | Which model, as a text id. The docs' examples currently use `"claude-opus-5-5"`. Smaller, faster, cheaper models suit a voice assistant; pick an id from https://platform.claude.com/docs/en/about-claude/models/overview |
| `max_tokens` | The longest reply you will accept. Spoken answers are short; a few hundred is plenty |
| `system` | Your standing instructions |
| `messages` | The conversation so far: a list of `{"role": "user" or "assistant", "content": ...}` |
| `tools` | Milestone 2 |

The Python package hides the HTTP details. `anthropic.Anthropic()` makes a client that reads `ANTHROPIC_API_KEY` from the environment by itself, and `client.messages.create(...)` takes the fields above. The reply has a `content` list of blocks; a block whose `type` is `"text"` has the words in `.text`.

`assistant.py`:

```
import anthropic
from dotenv import load_dotenv

load_dotenv()                      # reads .env into the environment
client = anthropic.Anthropic()     # finds the key there. Never pass it in by hand.

MODEL = "..."                      # choose from the models page

conversation = []                  # the running list of messages


def chat(user_text):
    # 1. add {"role": "user", "content": user_text} to the conversation
    # 2. call client.messages.create with model, max_tokens, system and messages
    # 3. add the assistant's reply to the conversation (role "assistant")
    # 4. pull the text out of the reply's content blocks and return it
    ...
```

In `main.py`, add:

```
class ChatRequest(BaseModel):
    # one field: the text the user said
    ...


@app.post("/chat")
def chat_endpoint(request: ChatRequest):
    # call assistant.chat and return {"reply": ...}
    ...
```

**Check:**

```bash
curl -X POST http://127.0.0.1:8000/chat -H "content-type: application/json" -d '{"text": "Hello, who are you?"}'
```

**Task 2: follow-ups.** Ask "What is the tallest mountain?" then "How tall is it in feet?" If step 3 of `chat` is done, the second answer makes sense. Remove step 3 for a moment and try again to see why it matters: the model remembers nothing between requests; your list *is* the memory.

**Task 3: the system prompt.** Put it in `prompts.py` as one string. Give the assistant a name and manner, say that replies are spoken aloud (so: short, no lists, no symbols), and say what it should do when it does not know.

**Task 4: speech on the phone.** A native plugin with the same shape as Stage 8. Design the C# side first:

```
public class SpeechBridge : MonoBehaviour
{
    public event System.Action<string> Heard;        // final text of what was said
    public event System.Action SpeakingFinished;

    public void RequestPermissions() { }
    public void StartListening() { }
    public void StopListening()  { }                 // triggers Heard
    public void Speak(string text) { }
}
```

On the Swift side, `SFSpeechRecognizer` turns microphone audio into text and `AVSpeechSynthesizer` reads text aloud; Apple's pages for both are linked above. Two permission messages are required in the app's Info.plist: `NSSpeechRecognitionUsageDescription` and `NSMicrophoneUsageDescription`. Without them the app closes the moment it asks.

Get the plugin working alone first: hold a button, speak, and show the recognised text on screen. No server.

**Tasks 5–6: join them up.** `AssistantClient.cs` has one method, `Send(string text, Action<string> onReply, Action onError)`, built exactly like Stage 11's `ApiClient`. `PushToTalk.cs` runs the sequence:

```
// button down  → state = Listening, speech.StartListening()
// button up    → speech.StopListening()
// Heard(text)  → state = Thinking,  client.Send(text, ...)
// reply        → state = Speaking,  speech.Speak(reply)
// SpeakingFinished → state = Idle
// any error    → say or show a short apology, state = Idle
```

**If it fails:**

| What you see | Likely cause |
|---|---|
| `AuthenticationError` or 401 | `.env` missing, misnamed, or the server was started from a different folder |
| The error mentions an unknown model | The id is misspelled or retired; copy one from the models page |
| A 400 about message roles | Two `user` messages in a row; the assistant reply was not added to the list |
| Follow-ups make no sense | The conversation list is being reset on each request |
| App closes when asking for the microphone | A usage description is missing from Info.plist |
| It hears its own voice | Listening restarted before speaking finished; wait for `SpeakingFinished` |

Commit after each task. Run `git status` first, every time.

### Milestone 2 — Tools

**Task 1: understand the loop first.** Verified against the tool-use page:

1. You send the conversation **plus a list of tool descriptions**.
2. If the model wants a tool, its reply has `stop_reason` equal to `"tool_use"` and a content block of type `"tool_use"` carrying an `id`, the tool's `name` and an `input` object.
3. **Your code** runs the tool. The model never runs anything itself.
4. You add the model's reply to the conversation, then add a `user` message whose content is a `tool_result` block carrying the matching `tool_use_id` and the result as text.
5. You call the API again. Now the model answers in words, or asks for another tool.

Steps 2–5 repeat until `stop_reason` is no longer `"tool_use"`. That is a `while` loop.

**Task 2: describe one tool.** A tool description has three parts: `name`, `description`, and `input_schema`, which lists each input's type and what it means. In `tools.py`:

```
TOOLS = [
    {
        "name": "...",
        "description": "...",     # when should the model use this? be specific
        "input_schema": {
            "type": "object",
            "properties": {
                # one entry per input: its type and a description
            },
            "required": [ ... ],
        },
    },
]


def run_tool(name, tool_input):
    # 1. which tool is it?
    # 2. call the matching Stage 11 function (the same code your endpoints use)
    # 3. return a short text result, e.g. "Timer set for 10 minutes"
    # 4. unknown tool or bad input? return a text error. Do not crash.
    ...
```

For step 2, move the body of each Stage 11 endpoint into a plain function that both the endpoint and the tool call. Then the HUD and the assistant cannot disagree.

**Task 3: put the loop in `chat`.**

```
def chat(user_text):
    # 1. add the user message
    # 2. loop:
    #      a. call the API with tools=TOOLS
    #      b. add the reply to the conversation
    #      c. if stop_reason is not "tool_use": leave the loop
    #      d. for each tool_use block: run_tool(...), collect a tool_result
    #      e. add ONE user message containing all the tool_result blocks
    # 3. return the text of the final reply
    # Safety: stop after, say, five trips round the loop.
    ...
```

**Check with curl before the phone:**

```bash
curl -X POST http://127.0.0.1:8000/chat -H "content-type: application/json" -d '{"text": "Set a timer for ten minutes"}'
```

```bash
curl http://127.0.0.1:8000/timers
```

The timer is in the list, and it is on your HUD.

**Tasks 4–6.** Add to-do and board-note tools one at a time. Then run the brief's breaking tests through `curl`, and print each tool call on the server so you can watch the model's decisions. For confirmations, a simple approach: the delete tool takes a `confirmed` input, and its description says to ask the user first.

**If it fails:**

| What you see | Likely cause |
|---|---|
| A 400 mentioning `tool_use` ids without `tool_result` | The tool result was not sent back, or its `tool_use_id` does not match |
| The model describes setting a timer but nothing happens | `tools` was not passed in the request, or the description is too vague |
| It never stops calling tools | Results are unclear; return plain confirmations, and keep the safety limit |
| Crash on odd input | `run_tool` assumed an input existed; check before using it |

### Milestone 3 — Vision

1. **Unity:** `ARCameraManager.TryAcquireLatestCpuImage` gives an `XRCpuImage`. Convert it to a texture's pixels, encode as JPEG (https://docs.unity3d.com/ScriptReference/ImageConversion.EncodeToJPG.html), and `Dispose()` the image straight away, as the manual page insists. Shrink it first: about 1,000 pixels on the long side is plenty.
2. **Sending it:** JSON carries text, so the image bytes are turned into text with `System.Convert.ToBase64String`. Add an optional `image` field to your `/chat` request.
3. **Server:** when an image is present, the user message's `content` becomes a list holding an image block and a text block. Copy the exact shape from the vision page linked above.
4. **Test without Unity** by having the server read a photo from disk, so you know the server half works before debugging the phone half.
5. Time it with and without an image, and at two sizes. Record the numbers in the log.

Do not store frames on the server unless you have decided to, and say so in `server/README.md`.

### Milestone 4 — Memory

Two different things, kept apart:

- **The conversation** is the `messages` list. Today it lives in a Python variable and vanishes on restart.
- **Facts** ("my chemistry exam is on the 14th") should outlast every conversation.

`memory.py`, built like `store.py`:

```
def remember(fact):          # add a line to memory.json
    ...

def recall_all():            # return every stored fact
    ...

def forget(fact_id):
    ...
```

Expose `remember` and `forget` as tools. For getting facts back in, start with the simplest thing: add all of them to the system prompt at the start of each conversation. Note at what point that stops being sensible.

Add `GET /memory` and `DELETE /memory/{id}` so you can see and remove what it knows. Add `memory.json` to `server/.gitignore`.

**Check:** tell it your exam dates. Restart the server. Ask what to revise this week.

### Milestone 5 — Wake word and Siri hand-off

1. **In the foreground:** keep the recogniser running and watch its partial results for your wake word. Only what follows the wake word is sent to the server. The recogniser can run on the device; look at the `requiresOnDeviceRecognition` option on Apple's speech pages.
2. **When the app is closed,** iOS will not let it listen. Hand off from Siri with an App Intent ([docs/apple-services.md](../../docs/apple-services.md)): Siri collects the question and opens your app with it.
3. **Interrupting:** `AVSpeechSynthesizer` has a stop method. Call it when the wake word or the button fires mid-reply.
4. **Measuring:** log a timestamp at each arrow in the diagram at the top of this guide. Find the largest gap before trying to fix anything. Streaming (linked above) lets speech start before the full reply has arrived.

## Quests: how to start

**Quiz me.** Start with a single text file of notes in a `private/` folder. Add two tools: `get_notes(topic)`, which returns the relevant section, and `record_result(question, correct)`. Splitting notes by heading is a good first answer to "which part to send". Get a three-question quiz working in `curl` before adding voice.

**Theme my board.** Needs the board's data on the server (Stage 10's *Sync* quest). Add a `get_board_notes` tool and a `set_note_groups` tool. Using a tool to receive the grouping means the result arrives as structured data, with no free text to pick apart.

**Presence.** `OrbView.cs` listens for `AssistantState` changes and switches look. Start with four colours, one per state. Then scale the orb with loudness: for the microphone see https://docs.unity3d.com/ScriptReference/Microphone.html. Reuse `PanelFollower` from Stage 11 to try each locking mode.

## Committing your work

```bash
git status
```

```bash
git add -A
```

```bash
git commit -m "Stage 12: milestone 2, timer tool works"
```

```bash
git push
```

Before every push in this stage, search the repo for your key's first few characters:

```bash
git grep -n "sk-ant" || echo "no key found"
```

If it prints a file name, do not push. Remove the key from the file, revoke it in the console, and create a new one.
