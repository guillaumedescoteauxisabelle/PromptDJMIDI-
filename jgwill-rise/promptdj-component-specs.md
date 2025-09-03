# PromptDJ MIDI: Component Specifications

This document provides a detailed specification for each major UI component in the PromptDJ MIDI application. The specification is designed to be language- and framework-agnostic, focusing on the creative intent, responsibilities, and functional logic required for a complete implementation.

---

### 1. `PromptDjMidi` (The Main Stage)

**Creative Intent**: To serve as the main stage for the user's creative session, organizing the musical palette, providing master controls, and offering ambient visual feedback on the complete soundscape.

**Responsibilities**:
-   Render a grid of `PromptController` components based on the application's main state (`Map<string, Prompt>`).
-   Manage and delegate master playback control (`play-pause` action).
-   Manage the UI state for MIDI device selection and MIDI CC visibility.
-   Serve as the central hub for event aggregation, listening for `prompt-changed` events from children and emitting a single, consolidated `prompts-changed` event.
-   Calculate and render the dynamic background gradient based on the weights and colors of all prompts.
-   Receive the master `audioLevel` and propagate it down to all child `PromptController` components.
-   Track and apply a `filtered` state to child prompts that have been rejected by the AI model.

**Properties (Inputs)**:
-   `initialPrompts: Map<string, Prompt>`: The set of prompts to display on first render.

**Internal State**:
-   `prompts: Map<string, Prompt>`: The single source of truth for the state of all prompts.
-   `playbackState: PlaybackState`: The current state of the audio engine, received from `LiveMusicHelper`.
-   `audioLevel: number`: The current audio output level (0.0-1.0), received from `AudioAnalyser`.
-   `showMidi: boolean`: A boolean flag to show or hide MIDI-related UI elements. Default: `false`.
-   `midiInputIds: string[]`: A list of available MIDI device IDs.
-   `activeMidiInputId: string | null`: The ID of the currently selected MIDI input device.
-   `filteredPrompts: Set<string>`: A set containing the `text` of prompts that the AI has filtered.

**Events (Outputs)**:
-   `prompts-changed`: Emits the entire `Map<string, Prompt>` when any prompt changes.
-   `play-pause`: Emits a signal to toggle the audio playback state.
-   `error`: Emits a user-friendly error message string.

---

### 2. `PromptController` (The Musical Idea)

**Creative Intent**: To be the interactive, physical representation of a single musical idea, allowing direct manipulation of its text, weight, and MIDI mapping.

**Responsibilities**:
-   Visually display the prompt's text, its weight (via `WeightKnob`), and its MIDI CC assignment.
-   Provide an interface for the user to edit the prompt's text content, with validation.
-   Handle user input to change the prompt's weight via direct manipulation (drag/wheel) of the `WeightKnob`.
-   Listen for MIDI CC messages from the `MidiDispatcher` and update its own weight if the incoming `cc` matches its assigned `cc`.
-   Manage a "learn mode" state to listen for and assign a new MIDI CC.
-   Visually indicate when a prompt has been filtered by the backend (e.g., dimmed color, different styling).

**Properties (Inputs)**:
-   `promptId: string`
-   `text: string`
-   `weight: number`
-   `color: string`
-   `cc: number`
-   `filtered: boolean`
-   `showCC: boolean`: Determines if the MIDI CC label is visible.
-   `midiDispatcher`: An object/service that emits `cc-message` events.
-   `audioLevel: number`: The master audio level, passed through to the child `WeightKnob`.

**Internal State**:
-   `learnMode: boolean`: A flag indicating if the component is currently waiting for a MIDI message to assign a new CC. Default: `false`.
-   `lastValidText: string`: Stores the previous text content for reversion on invalid input.

**Events (Outputs)**:
-   `prompt-changed`: Emits its complete `Prompt` data object whenever its state changes.

---

### 3. `WeightKnob` (The Control)

**Creative Intent**: To provide a tactile and visually responsive control for a prompt's influence, connecting the user's input directly to the sound and reflecting the music's energy.

**Responsibilities**:
-   Visually represent the current `value` (weight) as a radial indicator on a virtual knob.
-   Translate vertical mouse drag gestures and mouse scroll wheel events into changes in `value`.
-   Enforce the valid range for `value` [0.0, 2.0].
-   Display a "halo" effect behind the knob. The halo's size is determined by a combination of the knob's `value` and the incoming `audioLevel`.
-   The halo should be hidden when `value` is 0.

**Properties (Inputs)**:
-   `value: number`: The weight to display. Range: 0.0 to 2.0.
-   `color: string`: The color to use for the halo effect.
-   `audioLevel: number`: The normalized audio output level. Range: 0.0 to 1.0.

**Events (Outputs)**:
-   `input`: Emits the new `value` (number) whenever it is changed by the user.

---

### 4. `PlayPauseButton` (The Conductor's Baton)

**Creative Intent**: To provide a clear, unambiguous master control for starting, stopping, and loading the music—the primary action for the conductor of this AI orchestra.

**Responsibilities**:
-   Display one of three distinct icons: a "play" symbol, a "pause" symbol, or a "loading" spinner.
-   The displayed icon must be determined exclusively by the `playbackState` property.
-   Provide a large, easily clickable hit area that emits a generic `click` event.

**Properties (Inputs)**:
-   `playbackState: PlaybackState`: The state that determines which icon to show.
    -   `'stopped'` or `'paused'` -> Show Play Icon
    -   `'playing'` -> Show Pause Icon
    -   `'loading'` -> Show Loading Spinner

**Events (Outputs)**:
-   `click`: Emits a standard click event when the user interacts with it.
