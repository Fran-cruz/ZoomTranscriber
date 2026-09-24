# Zoom Transcriber Pro

**Zoom Transcriber Pro** is a Tampermonkey userscript that automatically captures live Zoom captions and turns them into a timestamped transcript.

It is designed for **Zoom Web Client meetings** and works directly from the captions displayed in the browser. The script automatically detects and enables Zoom captions when possible, watches the live caption DOM, reconstructs sentences as Zoom updates them, and provides two ways to work with the resulting transcript:

* 💾 **Auto-save** — periodically downloads the transcript as a `.txt` file.
* 📝 **Live Notes** — provides a floating panel where the live transcript and your own notes can be viewed together.

It also supports **keyword alerts**, including desktop notifications, audio alerts, and an on-screen warning whenever a configured keyword is detected.

---

## ✨ Features

* 🎙️ Automatically captures Zoom live captions
* ⚡ Handles captions that update word-by-word
* 🧩 Reconstructs changing caption lines into complete transcript entries
* ⏱️ Adds timestamps to every finalized line
* 💾 Automatically saves the transcript every **3 minutes**
* 📝 Floating live-notes panel
* 📋 Copy transcript only
* 📋 Copy transcript together with personal notes
* 🔔 Keyword detection
* 🔊 Three-beep audio alert when a keyword is detected
* 🖥️ Desktop notifications
* 🚨 On-screen keyword warning banner
* ⏳ Two-minute cooldown per keyword to prevent notification spam
* 🌐 Supports English and Spanish Zoom UI labels
* 🖼️ Detects captions inside supported same-origin iframes
* 🛡️ Filters common Zoom UI text to reduce false transcript entries
* 💾 Performs a final save when leaving/reloading the page

---

# 🚀 Setup

## 1. Install Tampermonkey

Install the **Tampermonkey** browser extension for your browser.

Then open the Tampermonkey dashboard.

## 2. Create the userscript

Create a new userscript and replace the default contents with the contents of:

```text
Zoom Transcriber Pro.user.js
```

Save the script.

The userscript is configured to run on Zoom Web Client pages:

```text
https://app.zoom.us/wc/*
https://*.zoom.us/wc/*
```

## 3. Open Zoom Web Client

Open your Zoom meeting through the browser.

Once the meeting loads, Zoom Transcriber Pro will start automatically.

You should eventually see the Transcriber control bar in the bottom-right corner of the meeting.

---

# ⚙️ Configuration

Most customization is done near the top of the script.

```javascript
const AUTO_SAVE_INTERVAL = 3 * 60 * 1000;
const KEYWORD_COOLDOWN   = 2 * 60 * 1000;
const FINALIZE_PAUSE     = 3000;

const KEYWORDS = [
    "add",
    "your",
    "keywords",
    "here"
];
```

## Auto-save interval

Controls how often the transcript is downloaded.

Default:

```javascript
const AUTO_SAVE_INTERVAL = 3 * 60 * 1000;
```

That is:

**3 minutes**

For example, to save every minute:

```javascript
const AUTO_SAVE_INTERVAL = 60 * 1000;
```

---

## Keyword cooldown

Controls how frequently the same keyword can trigger an alert.

Default:

```javascript
const KEYWORD_COOLDOWN = 2 * 60 * 1000;
```

That is:

**2 minutes per keyword**

For example:

```javascript
const KEYWORD_COOLDOWN = 30 * 1000;
```

would allow the same keyword to alert again after 30 seconds.

---

## Caption finalization delay

```javascript
const FINALIZE_PAUSE = 3000;
```

Zoom continuously modifies the same caption while someone is speaking.

For example:

```text
Hello
Hello everyone
Hello everyone and welcome
Hello everyone and welcome to the meeting
```

The script treats these as updates to the same line rather than four separate transcript entries.

When the caption stops changing for the configured amount of time, the line is finalized.

Default:

**3 seconds**

---

# 🔑 Keyword Alerts

Add the words or phrases you want to monitor to:

```javascript
const KEYWORDS = [
    "exam",
    "homework",
    "deadline",
    "important"
];
```

Keyword matching is case-insensitive.

For example, the keyword:

```text
deadline
```

can match:

```text
The deadline is Friday.
```

When a keyword is detected, the script can:

1. 🔊 Play three short beeps
2. 🖥️ Display a desktop notification
3. 🚨 Display a red warning banner inside Zoom

The notification includes the detected keyword and a shortened version of the caption.

### Cooldown

Each keyword has its own cooldown.

If `deadline` triggers an alert, seeing `deadline` repeatedly during the next two minutes will not generate additional alerts.

Other keywords can still trigger independently.

---

# 💾 Auto-save Mode

Click:

**💾 Auto-guardar**

The script switches to Auto-save mode and periodically downloads the captured transcript.

The filename follows this format:

```text
CONTROL-TRANSCRIPCION-DD-MM-YYYY.txt
```

For example:

```text
CONTROL-TRANSCRIPCION-23-09-2026.txt
```

The transcript uses timestamped lines:

```text
[20:14:03] Good evening everyone.
[20:14:11] Today we're going to review the assignment.
[20:15:27] The deadline is Friday.
```

The currently active caption may also be included with:

```text
…
```

to indicate that the line has not yet been finalized.

### Save manually

Click:

**⬇️ Guardar ahora**

to immediately download the current transcript.

The script also attempts to save the transcript when the page is unloaded.

---

# 📝 Live Notes Mode

Click:

**📝 Panel de notas**

A floating panel appears on top of Zoom.

The panel contains:

* finalized transcript lines
* the currently active caption
* your personal notes

The active caption is updated live while Zoom is still generating it.

Once the caption is finalized, it becomes part of the permanent transcript.

---

## ✍️ Adding notes

Type your note into the field at the bottom of the panel.

Press:

```text
Enter
```

to insert the note.

Use:

```text
Shift + Enter
```

to create a new line inside the note.

Your notes are visually separated from the transcript.

---

# 📋 Copying the transcript

The Notes panel provides two copy options.

### Solo transcripción

Copies only the captured transcript.

Example:

```text
[20:14:03] Good evening everyone.
[20:14:11] Today we're going to review the assignment.
```

### Con notas

Copies both transcript lines and your notes.

Personal notes are prefixed with:

```text
>
```

Example:

```text
[20:14:03] Good evening everyone.
> Ask professor about the assignment.

[20:14:11] Today we're going to review the assignment.
```

---

# 🖥️ Control Bar

The control bar appears in the bottom-right corner of the Zoom page.

It contains:

| Control           | Function                             |
| ----------------- | ------------------------------------ |
| 💾 Auto-guardar   | Enable Auto-save mode                |
| 📝 Panel de notas | Open/enable Live Notes mode          |
| ⬇️ Guardar ahora  | Immediately save the transcript      |
| 🟢 Indicator      | Shows that the transcriber is active |

The Notes panel can also be minimized and reopened using the floating 📝 button.

---

# 🎙️ How Caption Capture Works

Zoom does not necessarily provide each spoken sentence as a clean, static DOM element.

Instead, the same caption element can change repeatedly while someone speaks:

```text
Hola
Hola a todos
Hola a todos, bienvenidos
Hola a todos, bienvenidos a la reunión
```

Zoom Transcriber Pro tracks these changes and attempts to determine whether incoming text belongs to the current sentence.

The script uses several strategies:

### 1. Extension detection

If the new caption starts with the existing caption, it is treated as an extension.

### 2. Overlap detection

If Zoom replaces part of the caption, the script attempts to merge overlapping text.

### 3. Rollback detection

If Zoom temporarily displays a shorter version of the same caption, the script ignores it when the existing transcript already contains the text.

### 4. New sentence detection

If the incoming caption does not appear to belong to the active line, the current line is finalized and a new line begins.

### 5. Silence-based finalization

If the active caption stops changing for the configured `FINALIZE_PAUSE` period, it is finalized automatically.

---

# 🔍 Caption Detection

The script searches for Zoom caption elements using known Zoom selectors as well as generic fallbacks.

It checks selectors related to:

```text
caption
subtitle
transcript
```

It also checks supported same-origin iframes because Zoom may render caption content inside an iframe.

The observer uses a throttled `MutationObserver` so frequent DOM changes do not cause an excessive number of scans.

---

# 🤖 Automatic Caption Enabling

Zoom Transcriber Pro attempts to find the Zoom caption/transcription controls and activate them automatically.

It recognizes common labels such as:

```text
Mostrar subtítulos
Show captions
Show subtitles
Live transcript
Transcription
```

It also avoids controls that indicate captions are already being hidden or disabled.

Because Zoom's web interface can change over time, this is one of the parts of the script most likely to require maintenance if Zoom changes its DOM or button labels.

---

# 🛡️ UI Filtering

Zoom pages contain a lot of text that is not actually spoken dialogue.

The script filters common Zoom interface labels such as:

```text
Mute
Unmute
Start video
Participants
Chat
Settings
Leave meeting
Reactions
Mostrar subtítulos
Ocultar subtítulos
Silenciar
Compartir
Salir
Grabar
```

This helps prevent interface text from being accidentally added to the transcript.

---

# 📁 Transcript Format

Each finalized line follows:

```text
[TIMESTAMP] TEXT
```

Example:

```text
[20:31:02] Welcome everyone.
[20:31:08] Let's begin today's class.
[20:31:15] The first topic is statistical quality control.
```

Timestamps use the `es-HN` locale and include:

```text
HH:MM:SS
```

---

# 🔔 Notifications

The script requests browser notification permission when necessary.

When available, Tampermonkey's `GM_notification` API is used.

Otherwise, the script falls back to the browser's native Notification API.

If notifications are blocked, the in-page red keyword banner and audio alert can still provide an indication when supported.

---

# 🧰 Troubleshooting

## Captions are not being captured

Make sure:

1. You are using the **Zoom Web Client**.
2. Zoom live captions/transcription are available in the meeting.
3. The userscript is enabled in Tampermonkey.
4. The script matches the current Zoom URL.
5. Captions are actually visible in the Zoom interface.

Open the browser console and look for:

```text
[Zoom Transcriber]
```

The script logs important startup and caption-processing events there.

---

## The script does not automatically enable captions

Zoom's interface can change its HTML structure, classes, labels, or controls.

The script uses both direct selectors and text-based detection, but a future Zoom update may require changes to `findCaptionControl()`.

---

## Keyword alerts are not appearing

Check that your keywords are configured:

```javascript
const KEYWORDS = [
    "important",
    "deadline"
];
```

Also check:

* Browser notification permissions
* Tampermonkey permissions
* The keyword spelling
* The two-minute cooldown

The keyword matching is case-insensitive but otherwise uses direct substring matching.

---

## Transcript contains incorrect or duplicated text

Live caption reconstruction depends on the text Zoom exposes in the DOM.

The script already attempts to handle:

* expanding captions
* shortened captions
* overlapping captions
* word-wrap changes
* repeated DOM updates

However, caption recognition errors or unusual changes in Zoom's caption rendering can still result in imperfect transcript text.

---

# 🔐 Privacy

Zoom Transcriber Pro processes the captured caption text locally in the browser.

The script does **not** send the transcript to an external transcription service or remote API.

Transcript data is used for:

* local transcript reconstruction
* the on-page Notes panel
* browser downloads
* keyword detection
* browser/Tampermonkey notifications

The downloaded transcript is saved by your browser according to its normal download behavior.

---

# ⚠️ Important Notes

### Zoom Web Client only

The script targets Zoom's web client:

```text
app.zoom.us/wc
*.zoom.us/wc
```

It is not designed to capture captions from the native Zoom desktop application.

### Zoom UI changes

The script relies on Zoom's DOM structure and visible caption controls. Zoom interface updates may break automatic caption detection or caption extraction.

### Caption availability

The script can only capture text that Zoom exposes as live captions/transcription in the browser.

It does not independently listen to or transcribe microphone/system audio.

### Browser downloads

Auto-save works by creating a browser download every save cycle. Depending on browser download settings, repeated downloads with the same filename may be renamed automatically by the browser.

---

# 🧑‍💻 Development

The userscript is organized into several main components:

```text
Configuration
    ↓
Caption Engine
    ↓
Caption DOM Scanner
    ↓
Transcript State
    ├── Auto-save
    └── Live Notes
    ↓
Keyword Alerts
    ↓
Zoom UI Controls
```

### Main components

| Component                | Purpose                                         |
| ------------------------ | ----------------------------------------------- |
| `processCaption()`       | Processes incoming caption text                 |
| `isExtensionOf()`        | Determines whether text extends the active line |
| `overlapMerge()`         | Reconstructs overlapping caption updates        |
| `finalizeLine()`         | Commits the active caption to the transcript    |
| `scanCaptions()`         | Searches the page for caption text              |
| `startCaptionObserver()` | Watches Zoom's DOM for changes                  |
| `autoEnableCaptions()`   | Attempts to activate Zoom captions              |
| `startAutoSaveMode()`    | Enables periodic transcript downloads           |
| `startNotesMode()`       | Enables the live notes interface                |
| `checkKeywords()`        | Detects configured keywords                     |
| `triggerAlert()`         | Handles audio, notification, and banner alerts  |
| `createControlBar()`     | Creates the userscript controls                 |

---

# 📌 Current Defaults

| Setting                    |                    Default |
| -------------------------- | -------------------------: |
| Auto-save interval         |                  3 minutes |
| Keyword cooldown           |                  2 minutes |
| Caption finalization delay |                  3 seconds |
| Transcript format          |           Timestamp + text |
| Output format              |                     `.txt` |
| Timestamp locale           |                    `es-HN` |
| Caption scanning           |           MutationObserver |
| Keyword matching           | Case-insensitive substring |
| Target                     |            Zoom Web Client |

---

# 📜 License

No license is currently specified.

If you plan to distribute or publish the script, consider adding an explicit license such as MIT.

---

# 👤 Author

**Francisco Cruz**

Zoom Transcriber Pro
Version **6.0**

Built as a browser-side Tampermonkey tool for capturing and working with Zoom live captions.
