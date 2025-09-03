# PromptDJ MIDI: Creative Advancement Scenarios

This document outlines the key creative journeys enabled by the PromptDJ MIDI application, following the RISE framework's Creative Advancement Scenario format. These scenarios focus on the desired outcomes users can create, driven by the application's structural dynamics.

---

### Scenario 1: Crafting a Personalized Performance Instrument

**User Intent**: To create a personalized, tactile hardware instrument for live AI music performance, where each physical knob or fader corresponds to a specific musical idea.

**Current Structural Reality**: The user has a generic MIDI controller with unassigned controls and a grid of musical prompts on the screen. There is a disconnect between their physical hardware and the digital sound palette.

**Natural Progression Steps**:
1.  **Structural dynamic enables vision clarification**: The user activates "MIDI" mode, revealing the MIDI mapping interface. This signals the application's readiness to bridge the physical-digital gap.
2.  **System responds by inviting connection**: The user clicks the "CC" label on a `PromptController`. The controller enters a "Listening..." state, visually highlighting it and creating a clear target for the mapping action. This resolves the tension of "how do I connect this knob to that sound?"
3.  **User advances toward instrument creation**: The user turns a knob on their physical MIDI controller. The `MidiDispatcher` captures this signal.
4.  **System manifests the connection**: The `PromptController` exits learn mode, displaying the newly assigned CC number. The structural link is now formed.
5.  **User iterates to completion**: The user repeats this process, mapping more physical controls to on-screen prompts, progressively building out their custom instrument.

**Achieved Outcome**: The user has manifested a personalized MIDI instrument where their hardware is intuitively and directly connected to the AI's musical vocabulary, ready for a live performance.

**Supporting Features**:
-   `PromptController` (Learn Mode)
-   `MidiDispatcher`
-   `PromptDjMidi` (Show MIDI UI)

---

### Scenario 2: Performing a Live Sonic Blend

**User Intent**: To create a dynamic, evolving musical piece by performing a real-time mix of generative AI prompts.

**Current Structural Reality**: The user has a set of discrete musical ideas (prompts), each with a weight of 0 or 1. The music, if playing, is static or based on a simple combination.

**Natural Progression Steps**:
1.  **Structural dynamic enables creative expression**: The user manipulates a physical MIDI knob (or drags an on-screen `WeightKnob`). This action directly alters the `weight` property of a corresponding prompt.
2.  **System responds by updating its state**: The change in weight is communicated through the component hierarchy (`prompt-changed` -> `prompts-changed`). The visual feedback is immediate: the `WeightKnob` and background `halo` update.
3.  **User advances toward sonic creation**: The updated prompt weights are sent to the `LiveMusicHelper`. This resolves the tension between the user's desire to change the music and the AI's current output.
4.  **System manifests the new soundscape**: The `LiveMusicHelper` communicates with the generative music model, which seamlessly morphs the audio output to reflect the new blend of prompt weights.
5.  **User enters creative flow**: The user hears the musical change and reacts to it, adjusting other knobs to further sculpt the sound. A feedback loop of action, sound, and reaction is established, enabling a live performance.

**Achieved Outcome**: The user has created a unique, real-time musical performance, moving from a static set of ideas to a fluid and evolving soundscape.

**Supporting Features**:
-   `WeightKnob` (and MIDI mapping)
-   `LiveMusicHelper`
-   `AudioAnalyser` (providing visual feedback via `audioLevel`)
-   Dynamic Background Gradient

---

### Scenario 3: Curating the Musical Palette

**User Intent**: To create a custom musical palette by defining and refining the core sonic ideas the AI will use.

**Current Structural Reality**: The application presents a grid of default prompts (e.g., "Bossa Nova", "Chillwave"). These prompts may not align with the user's creative vision.

**Natural Progression Steps**:
1.  **Structural dynamic enables vision input**: The user focuses on a prompt's text label within a `PromptController`.
2.  **System responds by becoming editable**: The text field becomes an active input, allowing the user to type. This resolves the tension between the default palette and the user's desired sounds.
3.  **User advances toward a custom palette**: The user types a new prompt, such as "Cyberpunk Synthwave Bassline", and presses Enter or clicks away.
4.  **System manifests the new idea**: The `PromptController` updates its internal state and communicates the change. The `LiveMusicHelper` receives the new set of prompts, ready to incorporate this new musical idea into the generative mix.

**Achieved Outcome**: The user has created a bespoke musical palette that reflects their unique artistic vision, transforming the application from a generic tool into a personalized instrument.

**Supporting Features**:
-   `PromptController` (Editable Text)
-   Event propagation (`prompt-changed`, `prompts-changed`)
