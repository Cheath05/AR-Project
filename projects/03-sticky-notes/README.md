# Stage 3 — Anchors and persistence

**Scheduled:** see [docs/schedule.md](../../docs/schedule.md) · **Tools:** Unity 6, AR Foundation, your iPhone

> **New to coding?** Work through [GUIDE.md](GUIDE.md) alongside this brief. It says which files to create, where the code goes and how to check each step.

## What you will learn

- **Anchors:** points the tracker promises to keep fixed to the real world, even as its picture of the room changes.
- **Drift:** why un-anchored content slowly slides, and when you will notice.
- **Persistence:** saving the phone's map of a room and reloading it so content returns to the same real spot.
- **Serialisation:** turning your own data (note text, colour) into a file and back.

This is the stage that makes AR useful for studying: things you put somewhere are still there tomorrow.

## Read first

- AR Foundation, anchors: https://docs.unity3d.com/Packages/com.unity.xr.arfoundation@6.0/manual/features/anchors.html
- Apple on `ARWorldMap` (read the overview for the concept, not the Swift): https://developer.apple.com/documentation/arkit/arworldmap
- ARKit XR Plugin manual, the world map section, for the Unity API: https://docs.unity3d.com/Packages/com.unity.xr.arkit@6.0/manual/index.html
- `JsonUtility`: https://docs.unity3d.com/ScriptReference/JsonUtility.html

## Core build: sticky notes

**Goal:** stick text notes to your walls and desk. Quit the app, reopen it, and they are where you left them.

There is no starter. Create the Unity project inside this folder; you can begin from a copy of your Stage 2 project.

### Tasks

1. **See drift, then fix it.** Place one object the Stage 2 way and one attached to an anchor, side by side. Walk to another room and back. Is there a difference? Find out how an anchor is created at a raycast hit and what "attaching to a plane" adds.
2. **Notes on anchors.** Replace the object with a note: a coloured panel that sits flat against the surface it was placed on. A note on a wall and a note on a desk need different rotations. Where does that rotation come from?
3. **Editable text.** Tap a note to type into it with the iOS keyboard. Decide how the note looks while it is being edited and how you leave edit mode. Add a way to change its colour and delete it.
4. **Save.** Save two things when asked: the world map, and your own data for each note. The map knows about anchors but nothing about your text. How will you match a saved note to its anchor when they come back? Before you save, check the map's status; what does it tell you?
5. **Reload.** On launch, load the map and wait for the phone to recognise the room. Recreate each note on its anchor. What should the user see while the phone is still working out where it is, and what if it never does?

### Done when

- [ ] Notes sit flat on walls and tables and stay put as you move around.
- [ ] You can edit, recolour and delete a note.
- [ ] After a force-quit and relaunch in the same room, every note is back in place with its text.
- [ ] Relaunching in a different room does not crash or scatter notes at random.

### Check your understanding

Answer these in your own words in `docs/log.md`:

1. What does an anchor do that a plain world position does not?
2. What is in a world map? Why can it not be reloaded reliably if you rearrange the furniture or the lighting changes a lot?
3. Why not anchor everything? What is the cost?
4. Your notes are saved on one phone. What would have to change for a second phone to see them?

Question 4 comes back in Stage 9 (shared anchors) and Stage 5 (synced data).

## Study quest: Memory palace

**Scheduled:** see [docs/schedule.md](../../docs/schedule.md). Build it for a class starting the following week.

**Goal:** flashcards pinned along a route through your room, reviewed by walking it.

- A card has a front and a back. Tap to flip. What does your note data need that it did not have before?
- Order matters in a memory palace. Number the stops and draw a path between them so you know where to go next.
- Make the placement meaningful: put each card on an object you can link to the idea.
- Add a review mode that hides all the backs and tracks which ones you got right.
- Stretch: several palaces, one per subject, in the same room.

**Done when** you have walked the route twice on different days and the cards were in place both times.

## Study quest: Formula graffiti

**Scheduled:** backlog.

**Goal:** formulas written large on the wall above your desk, there every time you sit down.

- Big, readable text on a wall. How large does text need to be to read from a metre and a half away?
- Formulas need superscripts, fractions and symbols. Research what Unity's text component supports and where it runs out. What is your fallback for something it cannot draw?
- A "this week" set that you can swap without re-placing everything.
- Stretch: tap a formula to see what each symbol means, or a worked example.

**Done when** the wall holds the formulas for one real topic and reloads correctly.

## Study quest: Scale walks

**Scheduled:** backlog.

**Goal:** the solar system, or a historical timeline, laid out down your hallway at true relative scale.

- Pick a start point and a direction, then place items at distances worked out from real data.
- The real numbers will not fit in a hallway. Work out the scale factor. For the solar system, can distances and planet sizes share one scale? Try it.
- Label each stop with the real figure.
- Stretch: location anchors outdoors for a walk at a much bigger scale: https://developer.apple.com/documentation/arkit/arkit_in_ios/content_anchors/tracking_geographic_locations_in_ar

## Study quest: Quiz hunt

**Scheduled:** backlog.

**Goal:** questions hidden around the house; answer one correctly to learn where the next is.

- Each anchor holds a question, an answer and a clue to the next location.
- Only the current question is visible. How do you represent "where you are" in the hunt, and save it?
- Decide how answers are entered and checked. Multiple choice is the easy start.
- Stretch: a timer and a best time per hunt.

## Study quest: Focus pet

**Scheduled:** backlog.

**Goal:** a small creature anchored to your desk that grows while a study timer runs and sulks if you quit early.

- A timer, a pet with a few growth states, and a saved history so it remembers yesterday.
- What counts as quitting early? Research how a Unity app finds out it was sent to the background.
- Stretch: show the timer on the lock screen as a Live Activity. That is a Swift target outside Unity; start with [docs/apple-services.md](../../docs/apple-services.md).

## Play quest: Ghost replay

**Scheduled:** backlog.

**Goal:** record yourself walking a path through the room, then race a ghost that replays it.

- Record the camera's pose with a timestamp several times a second.
- Replay it on a ghost object. Samples are spaced apart in time; how do you move smoothly between them?
- Show who is ahead.
- Stretch: save the best run with the world map so it survives a relaunch.

## Play quest: Miniature weather

**Scheduled:** backlog.

**Goal:** a tiny cloud anchored over your desk showing the real weather outside.

- Start with fake data and three states: sun, cloud, rain. Research Unity's particle system for the rain.
- Then fetch real data. WeatherKit comes with your developer account; see [docs/apple-services.md](../../docs/apple-services.md). It has a REST API, so think about where the key should live.
- Stretch: wind direction tilts the rain; night dims the scene.

## Play quest: Geocache notes

**Scheduled:** backlog.

**Goal:** leave a message at an outdoor spot for a friend to find.

- Indoor anchors need a saved world map. Outdoors, ARKit can anchor to GPS coordinates in supported areas: https://developer.apple.com/documentation/arkit/arkit_in_ios/content_anchors/tracking_geographic_locations_in_ar
- Check whether your area is supported and what the fallback is when it is not.
- A note needs a latitude, a longitude and a message. How does your friend's phone get them?
- Stretch: notes that unlock only when you are within a few metres.

## If you get stuck

Ask for help in this order: concept explanation → hint → code review → minimal example. See [docs/roadmap.md](../../docs/roadmap.md#getting-help-without-getting-the-answer).
