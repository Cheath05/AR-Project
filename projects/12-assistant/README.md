# Stage 12 — Capstone: your own "Jarvis"

**Scheduled:** see [docs/schedule.md](../../docs/schedule.md) · **Tools:** Unity, your iPhone, your Stage 11 server, an LLM API account · **Builds on:** Stages 8, 10 and 11

> **New to coding?** Work through [GUIDE.md](GUIDE.md) alongside this brief. It says which files to create, where the code goes and how to check each step.

## What you will learn

- **Speech recognition and synthesis:** turning your voice into text and the reply back into sound.
- **LLM APIs:** sending a conversation to a model and handling what comes back.
- **Tool use:** letting the model ask your code to do things (start a timer, add a note) and return the result.
- **Keeping secrets server-side:** why the API key never leaves your server.
- **Conversation design:** what the assistant should do while it is listening, thinking and speaking.

## How realistic is this?

Very, for the useful version, on iPhone now and on a Quest 3 later. Every piece is available to one developer:

| Piece | What you use | Difficulty |
|---|---|---|
| Hearing you | Apple Speech on iPhone; Meta Voice SDK or Whisper on Quest | Easy |
| Thinking | An LLM over an API (e.g. Claude) with tool use, through your own server | Easy–medium |
| Speaking | Built-in text-to-speech or a hosted voice | Easy |
| Doing things | Tools you write (timers, calendar, board notes, quizzes) | Medium, grows forever |
| Seeing | A camera frame sent to a vision model | Medium |
| Memory | Your server stores notes and preferences | Medium |

**Limits to expect**

- Replies take 1–3 seconds. It will not feel instant.
- Always-on awareness is impractical on a headset: about 2 hours of battery, and streaming the camera continuously is slow and costs API money. Design for "ask when you need it".
- The assistant acts inside your own app and through web APIs. It cannot reach into other apps.
- API usage is pay-as-you-go, typically a few dollars a month at hobby levels. Set a spending limit in the provider's console before you start.

This brief fits any assistant persona. Name it, give it a voice and a personality of your own; if you are modelling it on a specific product or project, add a link to it here and adapt the milestones.

## Two rules

**1. The API key lives on your server. Never in the app, never in this repo.**
Anything inside an app can be extracted by someone who has the app. So the app talks to *your* server, and your server talks to the LLM. The key sits in an environment variable or an untracked `.env` file on the server (`.env` is git-ignored; commit a `.env.example` with a placeholder). If a key is ever committed, revoke it in the provider's console straight away; deleting the commit is not enough.

**2. The assistant is a service, not a scene.**
The conversation, tools and memory live on the server, next to your Stage 11 API. The Unity app captures audio, plays the reply and draws the orb. That keeps Stage 13 a port: the headset needs a new microphone and speaker hookup and nothing else.

## Read first

- Speech framework: https://developer.apple.com/documentation/speech
- Speech synthesis: https://developer.apple.com/documentation/avfoundation/speech-synthesis
- Native plug-ins for iOS (speech is a Swift API): https://docs.unity3d.com/Manual/PluginsForIOS.html
- Claude API docs: https://platform.claude.com/docs/en/home
- Tool use: https://platform.claude.com/docs/en/agents-and-tools/tool-use/overview
- Vision (for milestone 3): https://platform.claude.com/docs/en/build-with-claude/vision
- UnityWebRequest: https://docs.unity3d.com/ScriptReference/Networking.UnityWebRequest.html
- Your own `API.md` from Stage 11.

## Core build: five milestones

Each milestone leaves you with a working assistant that does a little more.

### Milestone 1 — Push-to-talk Q&A

**Goal:** hold a button, ask a question out loud, hear the answer.

1. Start on the server, with no phone involved. Add an endpoint that takes a text message and returns the model's reply. Test it from the command line. Where does the API key come from?
2. Keep the conversation going. The model has no memory between requests, so what must your server send each time for a follow-up question to make sense?
3. Write a system prompt that gives the assistant its name and manner, and tells it replies will be spoken aloud. How should a spoken answer differ from a written one?
4. On the phone, turn speech into text while a button is held. This needs a native plugin and two permission prompts. What happens if the user says no?
5. Send the text to your server and speak the reply.
6. Show the three states (listening, thinking, speaking) on screen, even as plain text for now.

**Done when:** you can ask two related questions in a row by voice and the second answer shows it understood the first.

### Milestone 2 — Tools

**Goal:** "set a timer for ten minutes", "add milk to my to-dos" and "put a note on my board" actually happen.

1. Read how tool use works until you can explain the loop in your own words: who decides to call a tool, who runs it, and what is sent back.
2. Describe one Stage 11 action (start a timer) as a tool: a name, a description and the inputs it takes. The description is what the model reads to decide when to use it, so write it carefully.
3. Handle the tool request on your server by calling your own API, then return the result to the model so it can answer.
4. Add tools for to-dos and for adding a note to a Stage 10 board. What does the board need to expose for this to work?
5. Try to break it: ask for something no tool covers, give half the information ("set a timer"), ask for two things at once. What should the assistant do in each case?
6. Which actions should it confirm before doing? Decide a rule for anything that deletes.

**Done when:** you ask out loud for a timer and a board note, and both appear in your HUD and on your board.

### Milestone 3 — Vision

**Goal:** "what am I looking at?" and "read this page" work.

1. Capture one camera frame while the AR session is running. See [image capture](https://docs.unity3d.com/Packages/com.unity.xr.arfoundation@6.0/manual/features/camera/image-capture.html).
2. Send it to your server along with the question. How large is the image, and how small can you make it before answers get worse? Measure the effect on reply time.
3. Decide when a frame is sent. Every question, or only when the question needs it? Who decides: your code, or the model through a tool?
4. Compare with Stage 8's on-device text recognition for "read this page". Which is faster, which is more accurate, which costs money?
5. Think about privacy: a frame can contain other people and their screens. When is it sent, and is anything stored?

**Done when:** you point the phone at an object or a textbook page, ask about it, and get a correct spoken answer.

### Milestone 4 — Memory

**Goal:** it remembers things across sessions: your classes, your preferences, what you asked it to remember.

1. Separate two kinds of memory: the current conversation, and facts that should outlast it. Where does each live?
2. Add "remember that…" and "what do you know about…" as tools backed by storage on your server.
3. Decide how remembered facts get back into a new conversation. All of them every time? Only relevant ones? What happens as the list grows?
4. Give yourself a way to see and delete everything it has stored.
5. Conversations get long. What do you do when one no longer fits in a single request?

**Done when:** you tell it your exam dates, quit the app, come back the next day and it uses them unprompted when you ask what to revise.

### Milestone 5 — Wake word and Siri hand-off

**Goal:** start talking to it without touching the phone.

1. Research how iOS limits listening in the background, and what that means for an always-listening wake word. What is possible while the app is open, and what is not once it is closed?
2. Add a wake word that works while the app is in the foreground. How do you avoid sending everything it hears to your server?
3. For when the app is closed, hand off from Siri with an App Intent: "Hey Siri, ask (your assistant's name)…". See [App Intents](https://developer.apple.com/documentation/appintents) and [docs/apple-services.md](../../docs/apple-services.md).
4. Let yourself interrupt it mid-sentence.
5. Measure the time from the end of your question to the start of the reply, and find the slowest step. Research streaming replies as one way to shorten it.

**Done when:** with the phone on your desk and the app open, you say the wake word, ask for a timer and get one, hands-free.

### Done when (whole core build)

- [ ] You ask out loud for a timer or a board note and it happens.
- [ ] It answers a question about something the camera can see.
- [ ] It remembers a fact from a previous day.
- [ ] The API key is only on the server, and `git log -p` shows no key was ever committed.
- [ ] The conversation, tools and memory run on the server; the Unity app handles audio and visuals only.
- [ ] A spending limit is set on the API account.

### Check your understanding

Answer these in your own words in `docs/log.md`:

1. Trace one request, "set a timer for five minutes", from your mouth to the spoken confirmation. List every hop and which machine it runs on.
2. Why can the key not live in the app, even in a private build only you use?
3. The model asked for a tool with an input your code did not expect. What happened, and what should happen?
4. Where does the 1–3 seconds go? Which part could you shorten, and how?
5. What would need to change for this to run on a Quest? What stays exactly the same?

## Study quests

### Quiz me

**Goal:** the assistant tests you from your own notes and PDFs, and keeps track of what you get wrong.

- Get your material to the server: text notes first, then PDFs.
- A whole textbook will not fit in one request. How will you choose which part to send for "quiz me on chapter 4"?
- Design the loop: it asks, you answer aloud, it judges and explains. How strict should the judging be for a spoken answer?
- Record results, and feed them into your Stage 5 spaced-repetition schedule so missed questions come back sooner.
- Ask it to quiz you without giving the answer away in the question. Test whether it does.

**Done when** you have done a ten-question spoken quiz on real course material and it correctly brought back a question you missed.

### Theme my board

**Goal:** it reads the post-its on a Stage 10 board and groups them by topic.

- Add a tool that returns a board's notes as data. This works only because the board's data is separate from its rendering.
- Ask the model for groups in a structured format your code can read. What format, and what do you do when the reply does not match it?
- Apply the result: move the notes into clusters, or colour them by group. Animate the move.
- Make it undoable. An assistant that rearranges your thinking needs an undo.

**Done when** you brainstorm fifteen messy notes, say "group these", and agree with most of the result.

## Play quest: Presence

**Goal:** a floating orb or character that makes the assistant feel like it is in the room.

- Give it a distinct look for idle, listening, thinking and speaking.
- Make it react to sound: pulse with the volume of your voice while listening, and with its own voice while speaking. See [Microphone](https://docs.unity3d.com/ScriptReference/Microphone.html).
- Where does it live? Try head-locked, world-locked and loose-follow from Stage 11. Which feels like a companion and which feels like a pop-up?
- Stretch: it turns to face whatever you are looking at when you ask a vision question.

## On a headset

The server does not change. On a Quest you swap Apple Speech for the Meta Voice SDK or Whisper, the iPhone camera frame for the Passthrough Camera API, and the orb moves from the phone screen into your room. Everything in "How realistic is this?" still applies, especially the limits.

## If you get stuck

Ask for help in this order: concept explanation → hint → code review → minimal example. See [docs/roadmap.md](../../docs/roadmap.md#getting-help-without-getting-the-answer).
