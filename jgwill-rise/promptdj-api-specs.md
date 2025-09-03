# PromptDJ MIDI: Internal API Specifications

This document outlines the internal event-based API of the PromptDJ MIDI application, focusing on how components communicate to enable creative advancement. This API is defined by custom events, which serve as the primary mechanism for state communication and advancing the application from user intent to musical manifestation.

---

### 1. `PromptController`

A single interactive unit for one musical prompt.

**Emits**:
-   `prompt-changed`
    -   **Payload**: `CustomEvent<Prompt>`
    -   **Detail**: An object conforming to the `Prompt` data model, containing the *complete and current state* of the prompt that was modified.
    -   **Creative Intent**: To signal that a core attribute of a single musical idea (its text, weight, or MIDI mapping) has been modified by the user.
    -   **Structural Intent**: This is the primary event for advancing the state of a single prompt. It allows the parent container (`PromptDjMidi`) to aggregate changes and maintain a complete picture of the desired soundscape without querying child components.
    -   **Triggered By**:
        -   User interaction (drag, wheel) with the `WeightKnob` results in a new weight value.
        -   Receiving a MIDI CC message matching its assigned `cc` number.
        -   The user finishes editing the prompt's text (on `blur` or `Enter` key).
        -   A MIDI learn operation successfully assigns a new `cc` number.

---

### 2. `PromptDjMidi`

The main component that orchestrates the grid of prompts and user controls.

**Listens For**:
-   `prompt-changed` (from `PromptController`)

**Emits**:
-   `prompts-changed`
    -   **Payload**: `CustomEvent<Map<string, Prompt>>`
    -   **Detail**: The complete, updated map of all prompts. The key is the `promptId` (string), and the value is an object conforming to the `Prompt` data model.
    -   **Creative Intent**: To communicate the entire, updated state of the musical palette to the audio engine.
    -   **Structural Intent**: This event resolves the tension between the user's interactions with individual prompts and the need for the `LiveMusicHelper` to have a holistic view of all prompt weights. It advances the application from a collection of individual states to a unified creative instruction.
    -   **Triggered By**: Immediately after handling a `prompt-changed` event from any child `PromptController`.

-   `play-pause`
    -   **Payload**: `CustomEvent<void>`
    -   **Detail**: No payload data. The event itself is the signal.
    -   **Creative Intent**: To give the user ultimate control over starting and stopping the musical creation.
    -   **Structural Intent**: A direct command to the `LiveMusicHelper` to toggle the master playback state, resolving the tension between silence and sound.
    -   **Triggered By**: User clicking the `PlayPauseButton`.

-   `error`
    -   **Payload**: `CustomEvent<string>`
    -   **Detail**: A string containing a user-friendly error message.
    -   **Creative Intent**: To inform the user of a technical barrier to their creative process (e.g., MIDI not available).
    -   **Triggered By**: `MidiDispatcher` failing to get MIDI access.

---

### 3. `LiveMusicHelper` (Service)

The service connecting the user's creative intent to the generative AI and audio output.

**Emits**:
-   `playback-state-changed`
    -   **Payload**: `CustomEvent<PlaybackState>`
    -   **Detail**: A string conforming to the `PlaybackState` type (`'stopped' | 'playing' | 'loading' | 'paused'`).
    -   **Creative Intent**: To provide clear visual feedback to the user about the current status of their musical creation.
    -   **Structural Intent**: This event advances the UI to match the audio engine's state, keeping the user informed and resolving uncertainty about whether the application is generating audio.

-   `filtered-prompt`
    -   **Payload**: `CustomEvent<LiveMusicFilteredPrompt>`
    -   **Detail**: The `LiveMusicFilteredPrompt` object received from the generative AI service.
    -   **Creative Intent**: To inform the user that one of their creative ideas could not be interpreted by the AI, allowing them to adjust their vision.
    -   **Structural Intent**: Resolves the discrepancy between the user's input and the model's capabilities by making the filtering process transparent.

-   `error`
    -   **Payload**: `CustomEvent<string>`
    -   **Detail**: A string containing a user-friendly error message.
    -   **Creative Intent**: To inform the user of a critical failure in the connection to the generative AI, explaining why their music has stopped.
    -   **Triggered By**: Connection errors or other fatal issues with the Lyria session.

---

### 4. `MidiDispatcher` (Service)

The utility for handling Web MIDI API interactions.

**Emits**:
-   `cc-message`
    -   **Payload**: `CustomEvent<ControlChange>`
    -   **Detail**: An object conforming to the `ControlChange` data model.
    -   **Creative Intent**: To translate a physical user action on a MIDI controller into a digital signal the application can use.
    -   **Structural Intent**: This is the core event that bridges the physical hardware and the digital software, resolving the primary tension for MIDI users and enabling the creation of a personalized hardware instrument.

---

### 5. `AudioAnalyser` (Service)

The utility for measuring audio output volume.

**Emits**:
-   `audio-level-changed`
    -   **Payload**: `CustomEvent<number>`
    -   **Detail**: A number representing the normalized audio level (from 0.0 to 1.0).
    -   **Creative Intent**: To provide a synesthetic visual representation of the music's volume, enhancing the user's sense of immersion and creative flow.
    -   **Structural Intent**: Creates a feedback loop from audio output back to the UI, allowing visual elements like the `WeightKnob` halo to react to the music being created.
