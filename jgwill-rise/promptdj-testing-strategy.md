# PromptDJ MIDI: Testing Strategy

This document outlines a testing strategy for the PromptDJ MIDI application based on the principles of the RISE framework. The focus is on validating that the application successfully enables users to achieve their desired creative outcomes through natural structural progression.

Testing should be oriented around verifying the **Creative Advancement Scenarios**, not just individual unit functionality.

---

### Guiding Principle

**Test for Creative Enablement, Not Just Correctness.**

Instead of asking "Does this function return the correct value?", we ask "Does this feature successfully advance the user toward their creative goal?"

---

### 1. Testing the "Personalized Instrument" Scenario

**Desired Outcome**: The user can successfully map MIDI CCs to on-screen prompts.

**Key Validations**:
-   **UI State Transitions**:
    -   Verify that clicking the "MIDI" button makes the CC labels and device dropdown visible.
    -   Verify that clicking a CC label puts the specific `PromptController` into a "learn mode" visual state (e.g., orange highlight, "Listening..." text).
    -   Verify that the controller returns to a normal state after a CC message is received.
-   **MIDI Message Handling**:
    -   Mock the Web MIDI API and the `MidiDispatcher`.
    -   Simulate a `cc-message` event being dispatched while a `PromptController` is in learn mode.
    -   Assert that the `PromptController`'s `cc` property is updated to match the message.
    -   Assert that a `prompt-changed` event is dispatched with the updated `cc` value.
-   **End-to-End Flow**:
    -   In an integration test, simulate the full user flow: click "MIDI", click a CC label, dispatch a mock MIDI event.
    -   Assert that the on-screen label now displays the new CC number.

---

### 2. Testing the "Live Sonic Blend" Scenario

**Desired Outcome**: User manipulation of controls results in a corresponding update to the prompt weights sent to the music engine.

**Key Validations**:
-   **UI Interaction to State Change**:
    -   Simulate a drag or wheel event on a `WeightKnob`.
    -   Assert that the knob's `value` property updates correctly and is clamped between 0 and 2.
    -   Assert that a `prompt-changed` event is dispatched with the new weight.
-   **MIDI Input to State Change**:
    -   Dispatch a mock `cc-message` event that matches a `PromptController`'s `cc` number.
    -   Assert that the controller's `weight` is updated based on the message's `value`. (e.g., `value / 127 * 2`).
    -   Assert that a `prompt-changed` event is fired.
-   **State Aggregation**:
    -   In a test of the `PromptDjMidi` component, simulate a `prompt-changed` event from a child controller.
    -   Assert that `PromptDjMidi` dispatches a `prompts-changed` event.
    -   Verify that the `Map<string, Prompt>` payload of this event correctly reflects the change.
-   **Engine Communication**:
    -   Mock the `LiveMusicHelper`.
    -   Trigger a `prompts-changed` event on the `PromptDjMidi` component.
    -   Assert that the `liveMusicHelper.setWeightedPrompts` method is called with the correct, updated map of prompts.

---

### 3. Testing the "Curating the Palette" Scenario

**Desired Outcome**: The user can edit the text of a prompt, and that change is propagated to the music engine.

**Key Validations**:
-   **Editable Text Interaction**:
    -   Simulate a `focus` event on a prompt's text element.
    -   Simulate user input (changing the `textContent`).
    -   Simulate a `blur` event.
-   **State Update and Event Dispatch**:
    -   Assert that the `PromptController`'s `text` property is updated to the new trimmed value.
    -   Assert that a `prompt-changed` event is dispatched with the new text.
-   **End-to-End Propagation**:
    -   As with the "Live Sonic Blend" scenario, verify that this change propagates through a `prompts-changed` event and results in a call to `liveMusicHelper.setWeightedPrompts`.
-   **Input Validation**:
    -   Test edge cases like entering an empty string or a string with only whitespace. Assert that the text reverts to its last valid state.
    -   Test the 'Escape' key behavior, asserting that it reverts the text and blurs the input.
