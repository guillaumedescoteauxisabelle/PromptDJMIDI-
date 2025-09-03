# PromptDJ MIDI: Data Models

This document specifies the core data structures used in the PromptDJ MIDI application, viewed through the lens of the RISE framework. These models represent the structural reality of the user's creative vision as it is understood by the application.

---

### 1. `Prompt`

This is the central data model, representing a single, atomic musical idea within the user's creative palette.

**Definition**:
```typescript
interface Prompt {
  readonly promptId: string; // A stable, unique identifier for the UI element.
  text: string;             // The user's creative instruction. Must be a non-empty string.
  weight: number;           // The influence of this idea. Range: 0.0 to 2.0.
  cc: number;               // The MIDI Control Change number. Range: 0 to 127.
  color: string;            // A visual identifier (e.g., CSS hex color).
}
```

**RISE Analysis**:
-   **Creative Intent**: The `Prompt` object is the digital manifestation of a user's musical idea. The `text` property holds the creative vision, while the `weight` property represents its current prominence in the overall composition. The `cc` property links this abstract idea to a physical, tactile control.
-   **Structural Intent**: This model encapsulates all the necessary information for a single `PromptController`. It provides the data needed to render the UI, process user input (both mouse and MIDI), and communicate changes.
-   **Advancement Intent**: Modifying the properties of a `Prompt` object (e.g., changing its `weight` or `text`) is the primary mechanism through which the user advances from their current sonic reality to their desired one.

---

### 2. `Map<string, Prompt>`

This data structure represents the user's entire musical palette—the collection of all `Prompt` objects currently active in the grid. It is the application's central source of truth for the creative state.

**RISE Analysis**:
-   **Creative Intent**: This map represents the user's complete creative vision at a given moment. It is the full set of instructions sent to the generative AI to create the soundscape.
-   **Structural Intent**: Using a `Map` with `promptId` as the key provides an efficient way to manage and update the collection of prompts. It allows for quick lookups and updates as `prompt-changed` events are received from individual controllers.
-   **Advancement Intent**: This collection is the payload of the `prompts-changed` event. Sending the entire map to the `LiveMusicHelper` ensures the audio engine has a complete and consistent state, advancing the musical output to perfectly match the user's full intention.

---

### 3. `ControlChange`

This model represents a raw message received from a physical MIDI device.

**Definition**:
```typescript
interface ControlChange {
  channel: number; // MIDI channel. Range: 0 to 15.
  cc: number;      // The controller number. Range: 0 to 127.
  value: number;   // The controller value. Range: 0 to 127.
}
```

**RISE Analysis**:
-   **Creative Intent**: Represents a single, discrete physical action taken by the user (e.g., turning a knob). It is the raw input of the user's performance gesture.
-   **Structural Intent**: This is the standardized format for MIDI CC data within the application. It decouples the `MidiDispatcher` from the `PromptController`, allowing the dispatcher to simply report what it "heard" without needing to know what will be done with that information.
-   **Advancement Intent**: This data packet is the critical first step in the chain of events that allows a physical action to advance the musical composition. It is the bridge from the physical world to the application's digital state.

---

### 4. `PlaybackState`

A simple string union type representing the state of the audio engine.

**Definition**:
```typescript
type PlaybackState = 'stopped' | 'playing' | 'loading' | 'paused';
```

**RISE Analysis**:
-   **Creative Intent**: Represents the answer to the user's fundamental question: "Is my creation making sound right now?" It provides clarity and confidence.
-   **Structural Intent**: A finite state machine for the audio engine's status. It allows the UI (specifically the `PlayPauseButton`) to accurately reflect the state of the `LiveMusicHelper`, ensuring consistency.
-   **Advancement Intent**: Changes in this state represent major transitions in the user's creative session—from ideation ('stopped') to performance ('playing') or buffering ('loading').
