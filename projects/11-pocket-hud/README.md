# Stage 11 — Capstone: personal HUD

**Scheduled:** see [docs/schedule.md](../../docs/schedule.md) · **Tools:** Unity, AR Foundation, your iPhone, a small server on your Mac · **Builds on:** Stages 2, 3 and 5

> **New to coding?** Work through [GUIDE.md](GUIDE.md) alongside this brief. It says which files to create, where the code goes and how to check each step.

## What you will learn

- **Head-locked vs world-locked UI:** content that follows your view, content that stays in the room, and the in-between ("body-locked", lazy follow) that most headsets actually use.
- **Your own API:** a small server that owns your data and answers requests over HTTP.
- **Client and server:** the AR app as one of several possible front ends to the same data.
- **Live data:** fetching, refreshing and coping when the network is slow or gone.

## The one rule: the server owns the data

Your timers, to-dos and calendar live in the server. The AR app asks for them and draws them. It does not keep its own separate copy of the truth.

This is what makes the rest of the roadmap work: the Stage 12 assistant changes the same data through the same API, the lock-screen widget reads it, and in Stage 13 the headset app is a new client for a server that does not change at all.

## Read first

- An overview of HTTP: https://developer.mozilla.org/en-US/docs/Web/HTTP/Overview
- A small web framework in a language you know. Either is fine: FastAPI (Python) https://fastapi.tiangolo.com/ or Express (JavaScript) https://expressjs.com/
- UnityWebRequest: https://docs.unity3d.com/ScriptReference/Networking.UnityWebRequest.html
- JsonUtility: https://docs.unity3d.com/ScriptReference/JsonUtility.html
- World-space UI: https://docs.unity3d.com/Packages/com.unity.ugui@2.0/manual/HOWTO-UIWorldSpace.html
- Anchors (for world-locked panels): https://docs.unity3d.com/Packages/com.unity.xr.arfoundation@6.0/manual/features/anchors.html

## Core build: a dashboard around you

**Goal:** a clock, timers, your to-do list and today's calendar, arranged in the space around you, showing real data from a server you wrote.

### Tasks

1. **Design the API.** On paper, before any code. What things exist (a to-do, a timer, an event)? What fields does each have? What requests does a client need: list them, add one, tick one off, start a timer? Write each as a method, a path, and an example request and response in JSON. Commit it as `API.md` in this folder. A timer is the interesting one: what does the server store so that every client shows the same time remaining?
2. **Run it locally.** Build the server so it does what `API.md` says, running on your Mac. Test every request from the command line or a browser before Unity is involved. Where is the data kept when the server restarts? Then reach it from your iPhone on the same Wi-Fi. What address does the phone use, and why is it not `localhost`?
3. **HUD panels.** In Unity, build a panel for each kind of data, filled with fake data for now. Design for a glance: how much text is readable at arm's length? Keep each panel a separate, reusable piece.
4. **Head-locked vs world-locked modes.** Give every panel three behaviours you can switch between: fixed to the view, fixed in the room, and following you loosely (it drifts back into view after you turn). Try each for the clock, the to-do list and a running timer. Which feels right for which, and which is tiring? Write your findings in the log.
5. **Live data.** Replace the fake data with requests to your server. How often should each panel refresh? Ticking a to-do in AR should change it on the server. What does the HUD show while a request is in flight, and when the server cannot be reached?

### Done when

- [ ] Your real to-dos and timers show in AR on your iPhone, served by your own server.
- [ ] A change made in AR is still there after relaunching the app.
- [ ] A change made outside the app (with a command-line request) appears in the HUD.
- [ ] Each panel can switch between head-locked, world-locked and loose-follow.
- [ ] The app stays usable with the server switched off, and says clearly that it is offline.
- [ ] `API.md` matches what the server actually does.
- [ ] No address, password or key that should be private is committed.

### Check your understanding

Answer these in your own words in `docs/log.md`:

1. Which locking mode did you choose for each panel, and why? What made head-locked content uncomfortable?
2. On a phone, "head-locked" really means "screen-locked". How will this differ on a headset, where the display covers your whole view?
3. Two clients start the same timer a second apart. What does your server do?
4. Why does the phone need your Mac's network address? What would need to change to use the HUD away from home?
5. List everything in the Unity project that would need rewriting for a headset. Is the server on the list?

## Play quest: Lock-screen twin

**Goal:** the same data as a home-screen widget and a Live Activity, so a running timer is visible without opening the app.

- A widget is a separate small Swift program added to the Xcode project Unity generates. Read [docs/apple-services.md](../../docs/apple-services.md) for how that fits with Unity, and the note there about Unity regenerating the Xcode project.
- WidgetKit: https://developer.apple.com/documentation/widgetkit · ActivityKit: https://developer.apple.com/documentation/activitykit
- The widget is one more client of your API. How does it get the data: straight from the server, or from something the app saved?
- Widgets refresh on a budget the system controls. What does that mean for a countdown? Research how a widget shows a timer that ticks without being refreshed.
- Start a Live Activity when a timer starts, and end it when the timer finishes.

**Done when** you start a timer in AR, lock the phone, and watch it count down on the lock screen.

## Looking ahead

In Stage 12 the assistant will call this API to set timers and add to-dos for you. As you finish this stage, check that every action a person can take in the HUD is also possible with a plain request to the server. Anything that only works by tapping a panel is something the assistant will not be able to do.

## If you get stuck

Ask for help in this order: concept explanation → hint → code review → minimal example. See [docs/roadmap.md](../../docs/roadmap.md#getting-help-without-getting-the-answer).
