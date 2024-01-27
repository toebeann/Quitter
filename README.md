# Quitter
A configurable, universal BepInEx 5 plugin to quit the game when returning to the main menu, for games where not doing so causes a world of hurt for devs and users alike.

## Why would I want this?
Modding can be a pain sometimes. Unity games modded with BepInEx often weren't _meant_ to be modded in this way, and often it is challenging for mods to properly clean up after themselves when the user returns to the main menu. Situations like these can cause all kinds of nasty bugs without warning, up to and including permanently corrupting save files. Quitter is one way<sup><small>[[†]](#-is-there-a-better-way)</small></sup> to avoid this particular class of issues.

## Modes
- **Auto Quit**: Automatically quits the game when returning to the main menu after a skippable countdown, which is rendered on-screen. Can be cancelled!

- **Remind Me**: Displays a dismissible alert reminding you to quit the game after returning to the main menu.

## How does it work?
It relies on Unity's built-in `SceneManager` events to monitor when a scene with a configurable, matching name is loaded, unloaded or made the active scene (configurable), and Unity's built-in IMGUI (Immediate Mode GUI) system for all in-game rendering, to ensure Quitter is as game-agnostic as possible. If your game doesn't use a scene for its main menu or doesn't fire the appropriate `SceneManager` events, this won't work for you.

Some details about how each mode operates:

### Auto Quit
When the event is fired for the configured scene name:

#### Configured threshold NOT yet met
Quitter renders an info bar near the top of the screen informing the user that the game will automatically quit when the event is fired `X` more times, where `X = Trigger.Threshold - HowManyTimesTheTriggerHasTriggered` (pseudocode).

The rendered Auto Quit info bar will include:
- a message letting the user know the game will automatically quit after it is triggered `X` more times,
- a button to disable Auto Quit by switching to Remind Me mode instead - when activated the bar will instead show info about how the user has switched to Remind Me mode with a button to undo,
- a button to display more info about _why_ it is a good idea to quit the game after returning to the main menu, and
- a button to dismiss the bar

The Auto Quit info bar will automatically dismiss itself after a short period of inactivity.

The Auto Quit info bar will also be rendered when the game first loads, without affecting/considering the threshold.

#### Configured threshold HAS been met/exceeded
Quitter will display a configurable countdown prompt in the middle of the screen with a notice letting the user know that Quitter is going to automatically quit the game.

The rendered Auto Quit countdown prompt will include:
- a notice that Quitter is going to automatically quit the game,
- a countdown to when the game will automatically quit,
- a button to skip the countdown and quit immediately, and
- a button to cancel, preventing the automatic quit and dismissing the prompt.

If Auto Quit is cancelled, Quitter will revert to displaying [the Auto Quit info bar](#configured-threshod-not-yet-met).

### Remind Me
Quitter renders a persistent Reminder info bar near the top of the screen letting the user know that they should quit the game.

The rendered Remind Me info bar will include:
- a messge reminding the user that they should quit the game ASAP,
- a button to quit immediately,
- a button to display more info about _why_ it is a good idea to quit the game, and
- a button to dismiss the bar

The Remind Me info bar will display persistently until either dismissed manually or the game is quit.

## Configuration
Configurable with Configuration Manager, or by hand-editing the .cfg:

| Option | Default | Description |
| --- | --- | --- |
| **General.Enabled** | `true` | Whether Quitter is active at all. Disabling at runtime will unhook all events and dismiss any currently rendering notices. Re-enabling at runtime will rehook all events, with internal variables reset. |
| **General.Mode** | `AutoQuit` | The Quitter mode to operate under. Either `AutoQuit` or `RemindMe`. |
| **Trigger.SceneName** | `"MainMenu"` | The name of the game scene which should trigger Quitter. |
| **Trigger.SceneEvent** | `activeSceneChanged` | The event of Unity's `SceneManager` which should trigger Quitter. A combination of `activeSceneChanged`, `sceneLoaded` and `sceneUnloaded`. |
| **Trigger.SceneMode** | `Single \| Additive` | The Unity `LoadSceneMode`(s) which should trigger Quitter. A combination of `Single` and `Additive`. |
| **Trigger.Threshold** | `2` | The minimum number of times the scene event needs to fire before triggering Quitter. |
| **AutoQuit.CountdownSeconds** | `5` | The number of seconds to countdown before automatically quitting in `AutoQuit` mode. Has no effect in `RemindMe` mode. |
| **Shortcuts.Dismiss** | `Shift + Escape` | A convenience keybind to dismiss any on-screen notices currently being rendered by Quitter. |
| **Info.WhyShouldIQuit** | Too long to put here | An info text letting users know _why_ they should quit to desktop after backing out to the main menu. Requires advanced mode in Configuration Manager. |

## Hang on a minute, there's no code or binaries available in this repo!
Yeah, that's because this is currently just me brainstorming a proposal for how to handle these annoying problems, primarily for usage with Subnautica and Subnautica: Below Zero, but could likely be used in many games, especially if an API is added which allows triggering Quitter programmatically from other plugins.

My plan is to build a prototype to see how it feels, and once we've got something we like, it'll likely be released standalone at least to begin with. Depending on how people feel it may eventually be added to the BepInEx packs for SN1 and BZ.

I'll probably get around to actually start coding a prototype over the next week or so.

If you have feedback or suggestions about any of this, feel free to open an [issue](https://github.com/toebeann/Quitter/issues) or [pull request](https://github.com/toebeann/Quitter/pulls). I only ask that you please be respectful.

## Footnotes

### <sup>[†]</sup> Is there a better way?
In a perfect world, mods wouldn't get released without their developers testing and fixing every single edge case, including what happens when a user returns to the main menu and then starts a new game/loads a save. Unfortunately, we are not in such a utopia. This plugin will ideally help to raise developer awareness so that more mods might get released which _do_ consider these edge cases, but more realistically, devs will instead prioritise more important things (like making their mods work at all, and eating snacks), and who can blame them?

It's worth mentioning that I also considered making a system where devs can flag their mods as supporting this kind of edge case (without any additional dependencies for the mod), and then having the plugin auto-quit/prompt to quit only when mods which do not flag support are found installed, which would help to raise developer awareness of the issue and _potentially_ lead to mods addressing these edge cases, but this approach raises some concerns:
- mods can potentially just flag support even if they don't actually take care of these edge cases, unless we have a verification system where community members whitelist mods after they are flagged, which is an annoying kind of gatekeeping that just creates more work for everyone, and
- mods which haven't yet flagged support could end up getting hounded by users, creating a negative perception around those mods and their devs, which isn't great for community morale.

So, overall it seems better to suppress the problem by shifting the onus to the users, having them quit to desktop when returning to the main menu in a modded game, or at the least informing them that they _should_ do this, and what they risk by not doing so. The approach taken by Quitter (giving up) will almost certainly ensure that there will always be an abundance of mods which don't consider these edge cases, but that would be true regardless of whether Quitter exists or not. Quitter _will_ however guarantee<sup><small>[[††]](#-where-are-my-cheesey-snacks)</small></sup> more cheesey-snack time for everyone, so I'd say that's a win.

### <sup>[††]</sup> Where are my cheesey snacks?
Not a real guarantee.
