# Stage 5 — Capstone: study buddy

**Scheduled:** Feb 8 – Mar 7, 2027 · **Tools:** Unity 6, AR Foundation, Xcode, a little Swift · **Builds on:** stages 2, 3 and 4

## What you will learn

- **App architecture:** turning several separate experiments into one app with shared data and a way to move between modes.
- **Data model:** describing subjects, cards and progress as plain data, separate from anything that draws them.
- **Sync:** getting the same data onto more than one device, and what can go wrong when two copies disagree.

This is the first stage with no new AR concept. The hard part is design.

## Read first

- Your own log notes from the Flip cards, Memory palace and Textbook triggers quests. What annoyed you is your requirements list.
- JsonUtility: https://docs.unity3d.com/ScriptReference/JsonUtility.html
- World-space UI: https://docs.unity3d.com/Packages/com.unity.ugui@2.0/manual/HOWTO-UIWorldSpace.html
- How Apple services fit with Unity: [docs/apple-services.md](../../docs/apple-services.md)
- CloudKit overview (skim; you decide in task 5 whether to use it): https://developer.apple.com/documentation/cloudkit

## Core build: one app for studying

**Goal:** a single app with a subject menu that launches the study quests you have already built, remembers your progress, and that you actually use before midterms.

There is no starter. Create the Unity project in this folder and copy in scripts from earlier stages as you need them.

### Tasks

1. **Sketch it on paper.** Draw every screen and the arrows between them before opening Unity. Which parts are flat 2D UI and which are AR? Where does the user land on launch? Photograph the sketch into this folder.
2. **Design the data model.** Write plain C# classes (no `MonoBehaviour`) for a subject, a card and a record of how well you know that card. What does every quest need in common? What is specific to one quest? Check your model by writing three real cards from a real class as JSON by hand.
3. **Subject menu and the first mode.** Build the menu from your data, not hard-coded buttons. Selecting a subject and a mode should start *Memory palace* showing only that subject's cards. How will the AR mode find out which subject was chosen?
4. **Bring in two more modes.** Add *Textbook triggers* and one Stage 2 quest. You will find code that assumed it was the only thing in the app. What has to change so three modes can share one AR session, and how do you clean up when leaving a mode?
5. **Save, then sync.** First save progress to a local file and reload it on launch. Then get it onto a second device. Two accepted routes; pick one and write down why:
   - **iCloud / CloudKit.** There is no Unity wrapper, so this means writing a small Swift native plugin that C# calls. More work, teaches you the bridge you need again in Stage 8.
   - **Your own small server.** A simple HTTP endpoint that stores and returns your JSON. Less Apple-specific, and the same server grows into Stage 11.

   Either way: what happens if you study on two devices before they sync?

### Done when

- [ ] A subject menu, generated from data, launches at least three study modes.
- [ ] Adding a new card means editing data, not code.
- [ ] Progress survives a force-quit and appears on a second device (or a second install).
- [ ] You have used it for a real study session and logged what to fix.
- [ ] A TestFlight build is installed on your phone.

### Check your understanding

Answer these in your own words in `docs/log.md`:

1. Why keep the data classes free of `MonoBehaviour`? What does that buy you in Stage 13?
2. Two devices changed the same card's progress offline. Which one wins in your design, and is that the right answer?
3. Which of your earlier scripts were hardest to reuse, and what would you do differently when writing the next one?
4. What is stored per device (anchors, world maps) and what is stored per user (cards, progress)? Why can't the first kind simply be synced?

## Study quest: Spaced-repetition room

**Scheduled:** Feb 22 – Feb 28.

**Goal:** cards you are due to review today glow or float closer; cards you know well fade into the background.

- Read about spaced repetition and SM-2: https://en.wikipedia.org/wiki/SuperMemo
- Write the scheduler first as plain C# with no AR and no Unity scene: given a card's history and how well you just answered, when is it next due? Test it by feeding it made-up answers and printing the dates.
- Only then connect it to the room. What visual difference between "due" and "not due" is obvious at a glance without being distracting?
- How do you record an answer in AR? Tap, swipe, something else?
- Stretch: a summary panel on the wall showing how many cards are due per subject.

**Done when** the app tells you what to study today and that list changes correctly the next day.

## Study quest: Study reminders

**Scheduled:** Mar 1 – Mar 7.

**Goal:** a notification when cards are due.

- Start with **local** notifications, scheduled on the device. They need no server and no push entitlement. Docs: https://developer.apple.com/documentation/usernotifications
- From Unity this is a native call. Research the options: a Unity package for mobile notifications, or your own Swift plugin. Which is less work, and which teaches you more?
- When should the reminder fire, and how do you avoid nagging on a day with nothing due?
- Stretch: true remote push, sent from your server. What extra pieces does that need?

**Done when** a reminder arrives on a day with due cards and tapping it opens the right subject.

## Study quest: Siri shortcuts

**Backlog.**

**Goal:** "Hey Siri, quiz me on chapter 4" opens the app in the right mode.

- App Intents are Swift, and they must live in the Xcode project Unity generates. Docs: https://developer.apple.com/documentation/appintents
- Unity regenerates that project on each build. Read the "Setting up a capability" section of [docs/apple-services.md](../../docs/apple-services.md) and decide how your Swift file will survive a rebuild.
- How does the intent tell the running Unity scene which subject to open?

**Done when** one spoken phrase starts a quiz without touching the screen.

## If you get stuck

Ask for help in this order: concept explanation → hint → code review → minimal example. See [docs/roadmap.md](../../docs/roadmap.md#getting-help-without-getting-the-answer).
