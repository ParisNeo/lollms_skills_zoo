---
name: video-builder
description: Builds self-contained HTML "Video Studio" files that animate concepts and narrate them via Text-to-Speech, allowing the user to record and export an educational video. Use when the user wants to create an animated explainer video, educational video, or tutorial from documentation.
when_to_use: "create video, make explainer, build tutorial, animated video, educational video, video studio, record presentation, text-to-speech video"
argument-hint: "[topic or path to documentation]"
---

# Video Builder Skill

## Overview
This skill creates self-contained HTML files that act as "Video Studios." The HTML file contains animated slides, a Text-to-Speech (TTS) narration engine, and a screen recording pipeline that allows the user to export a `.webm` video file with one click.

### Execution Workflow

When invoked, follow these steps in order. Do NOT skip steps.

#### Step 1: Gather User Style & Preferences
Ask the user (via `<lollms_form>`) for:
1.  **Video Topic**: The main subject.
2.  **Tone**: (e.g., Academic, Casual, Energetic).
3.  **Catchphrase**: A signature opening or closing line.
4.  **Visual Style**: (e.g., Dark/Cyberpunk, Light/Minimalist, Nature).
5.  **Documentation**: Ask the user to provide text, notes, or documents. If they provide files, read them. If they provide text, use it directly.

#### Step 2: Process Documentation
1.  Analyze the provided documentation.
2.  Break the content into **8 to 12 logical slides**.
3.  For each slide, write:
    *   A **Title** (hidden during recording).
    *   **Bullet points** or key concepts (hidden during recording).
    *   A **Narration Script** (2-4 sentences, conversational, incorporating the user's tone and catchphrase).
    *   An **Animation Concept** (what the canvas should draw).
4.  Save this breakdown as a Markdown artifact named `video-script.md` so you have a persistent reference.

#### Step 3: Generate the HTML Video Studio
Create a single HTML artifact named `video-studio.html`. The file MUST contain:

**A. HTML Structure**
*   `#stage`: Container for slides.
*   `.slide`: Individual slide divs (10-12 total).
*   `#caption-bar`: Fixed bottom bar for narration text.
*   `#control-panel`: Fixed bottom panel with Voice select, Speed slider, Record button.
*   `#file-warning`: A modal warning if opened via `file://`.

**B. CSS Styling**
*   **Theme**: Apply colors based on user's visual style choice.
*   **`body.recording`**: A class that sets `opacity: 0` for `.slide-title`, `.slide-subtitle`, `.equation`, `.info-row`, `.feature-list`, `.law-table`. This is **CRITICAL** – text must not appear in the video.
*   **Canvas**: Ensure `canvas` elements are centered and responsive.

**C. JavaScript Engine**
*   **`NARRATION` Array**: Array of strings containing the narration for each slide.
*   **TTS Engine**: Use `window.speechSynthesis`. Implement `speak(text)` returning a Promise.
*   **Recording Engine**:
    *   Use `navigator.mediaDevices.getDisplayMedia({ video: true, audio: true })`.
    *   Instruct user to enable "Share tab audio".
    *   Fallback: `audioDest = audioContext.createMediaStreamDestination()`.
    *   Merge tracks: `new MediaStream([...videoTracks, ...audioTracks])`.
    *   Use `MediaRecorder` to capture the stream.
    *   On stop, create a Blob and trigger download (`video.webm`).
*   **Animation Engine**: Use `requestAnimationFrame` for each slide's canvas animation. Animations must loop infinitely.

#### Step 4: Verify Content & Physics (Reflection)
Before finalizing, you MUST verify the content of the animations against the `video-script.md` and the source documentation.
*   **Check 1**: Are the visual elements representing the correct concepts? (e.g., If showing magnetic poles, is N facing S? If showing a solenoid, does the right-hand rule match the current direction?)
*   **Check 2**: Is the narration script accurate to the source material?
*   **Check 3**: Are there any contradictory visual elements?
*   If errors are found, fix them in the HTML code before presenting to the user.

#### Step 5: Final Output
Provide the user with:
1.  The `video-studio.html` file.
2.  Instructions on how to use it:
    *   Start a local server (`python -m http.server 8080`).
    *   Open `http://localhost:8080/video-studio.html`.
    *   Select voice, click "Start Recording".
    *   Enable "Share tab audio" in the browser dialog.
    *   Wait for the video to finish and auto-download.
    *   Convert to MP4 (optional): `ffmpeg -i video.webm -c:v libx264 video.mp4`.

### Critical Constraints
*   **No Text in Video**: The `body.recording` class must hide all text elements.
*   **Audio Capture**: The recording engine must attempt to capture tab audio, falling back to Web Audio API destination.
*   **Self-Contained**: The HTML must be a single file with no external dependencies (except Google Fonts).
*   **Accuracy**: Visuals must match the physics/concepts being taught. Verify before outputting.
*   **Looping Animations**: Canvas animations must run continuously via `requestAnimationFrame`.
