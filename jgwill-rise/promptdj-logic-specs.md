# PromptDJ MIDI: Core Logic Specifications

This document specifies the core business logic, calculations, and rules that govern the behavior of the PromptDJ MIDI application. This logic is intended to be language- and framework-agnostic.

---

### 1. MIDI Value to Prompt Weight Conversion

**Intent**: To map a standard MIDI CC value to the application's internal prompt weight scale.

-   **Input**: `midiValue`, an integer.
-   **Output**: `promptWeight`, a float.
-   **Formula**: `promptWeight = (midiValue / 127) * 2`
-   **Constraints**:
    -   Input `midiValue` must be clamped to the range [0, 127].
    -   The output `promptWeight` will be in the range [0.0, 2.0].

---

### 2. Prompt Text Input Validation

**Intent**: To ensure prompts are always non-empty, meaningful strings and to provide intuitive user input handling.

-   **State**: Each `PromptController` must maintain its `lastValidText`.
-   **On Focus**: When the text input gains focus, its entire content should be selected.
-   **On Blur (Loss of Focus)**:
    1.  Read the current text content from the input element.
    2.  Trim leading and trailing whitespace from the text.
    3.  **If** the trimmed text is an empty string:
        -   Revert the input element's content to `lastValidText`.
        -   Do not dispatch a `prompt-changed` event.
    4.  **Else**:
        -   Update the component's internal `text` state with the new trimmed text.
        -   Update `lastValidText` to this new value.
        -   Dispatch a `prompt-changed` event with the updated state.
-   **On `Enter` Key Press**:
    -   Prevent the default newline action.
    -   Trigger the same logic as the "On Blur" event.
    -   Remove focus from the text input element.
-   **On `Escape` Key Press**:
    -   Prevent any default action.
    -   Revert the input element's content to `lastValidText`.
    -   Remove focus from the text input element.
    -   Do not dispatch a `prompt-changed` event.

---

### 3. Dynamic Background Gradient Calculation

**Intent**: To create an ambient, synesthetic visual feedback of the current musical mix.

-   **Input**: `prompts: Map<string, Prompt>`.
-   **Output**: A string suitable for a CSS `background-image` property (e.g., `radial-gradient(...)`, `radial-gradient(...)`).
-   **Constants**:
    -   `MAX_WEIGHT_FOR_ALPHA = 0.5` (The prompt weight at which the gradient has maximum opacity).
    -   `MAX_ALPHA = 0.6` (The maximum opacity).
    -   `GRID_WIDTH = 4` (Number of columns in the prompt grid).
-   **Logic**:
    1.  Initialize an empty array `gradients`.
    2.  Iterate through the input `prompts` with an index `i`. For each `prompt`:
        a.  Calculate the alpha percentage: `alphaPct = clamp(prompt.weight / MAX_WEIGHT_FOR_ALPHA, 0, 1) * MAX_ALPHA`.
        b.  Convert `alphaPct` to a two-digit hexadecimal string (`alphaHex`).
        c.  Determine the gradient's origin position based on its grid index:
            -   `x = (i % GRID_WIDTH) / (GRID_WIDTH - 1)` (normalized 0 to 1)
            -   `y = floor(i / GRID_WIDTH) / (GRID_WIDTH - 1)` (normalized 0 to 1)
        d.  Calculate the gradient's size (radius stop point) based on its weight: `stop = prompt.weight / 2` (normalized 0 to 1, representing 0% to 100%).
        e.  Construct a radial gradient string: `radial-gradient(circle at ${x*100}% ${y*100}%, ${prompt.color}${alphaHex} 0px, ${prompt.color}00 ${stop*100}%)`.
        f.  Add the generated string to the `gradients` array.
    3.  Join the `gradients` array with `", "`.
-   **Performance**: This calculation should be throttled to execute no more frequently than once every 30 milliseconds.

---

### 4. `WeightKnob` Halo Effect Calculation

**Intent**: To provide immediate, per-prompt feedback on its weight and the overall audio output level.

-   **Input**: `promptWeight` (0.0-2.0), `audioLevel` (0.0-1.0).
-   **Output**: `scale`, a float representing the CSS transform scale factor.
-   **Constants**:
    -   `MIN_HALO_SCALE = 1.0` (Scale at weight 0).
    -   `MAX_HALO_SCALE = 2.0` (Maximum scale from weight alone).
    -   `HALO_LEVEL_MODIFIER = 1.0` (How much the audio level affects the scale).
-   **Logic**:
    1.  **If** `promptWeight` is 0, the halo should be hidden.
    2.  Calculate the base scale from weight: `scaleFromWeight = MIN_HALO_SCALE + (promptWeight / 2) * (MAX_HALO_SCALE - MIN_HALO_SCALE)`.
    3.  Calculate the final scale including audio level: `finalScale = scaleFromWeight + (audioLevel * HALO_LEVEL_MODIFIER)`.
    4.  Return `finalScale`.
