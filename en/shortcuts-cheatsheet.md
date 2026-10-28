# Burp Suite Keyboard Shortcuts Cheat Sheet

> 🌐 **Language / اللغة:** [النسخة العربية (`ar/shortcuts-cheatsheet.md`)](../ar/shortcuts-cheatsheet.md)

---

## 🎯 Appendix Objectives

Provide an immediate, authoritative desk reference covering essential **Keyboard Shortcuts (Hotkeys)** across **Windows / Linux** and **macOS** environments. Mastering these operational shortcuts dramatically accelerates manual request tampering and eliminates reliance on GUI mouse clicks during time-sensitive penetration tests and CTF competitions.

---

## 🚀 1. Cross-Tool Routing & Navigation

These shortcuts function globally across any Burp Suite tab (Proxy History, Scanner, Logger, Target):

| Action / Capability | Windows / Linux | macOS | Operational Notes |
| :--- | :---: | :---: | :--- |
| **Send Request to Repeater** | `Ctrl + R` | `⌘ + R` | The most frequent action in Burp; isolates a transaction for manual probing. |
| **Send Request to Intruder** | `Ctrl + I` | `⌘ + I` | Routes the transaction to configure fuzzing markers and wordlists. |
| **Archive Transaction to Organizer** | `Ctrl + O` | `⌘ + O` | Freezes an immutable proof-of-concept (PoC) snapshot for reporting. |
| **Send Request to Sequencer** | `Ctrl + S` | `⌘ + S` | Dispatches live token harvesting for cryptographic entropy auditing. |
| **Cycle Adjacent Tabs** | `Ctrl + Tab` | `Ctrl + Tab` | Fast tab navigation across open Repeater or Intruder workspaces. |
| **Close Active Tab** | `Ctrl + W` | `⌘ + W` | Closes the selected Repeater request tab. |

---

## 🛡️ 2. Proxy Interception & Flow Control

Operated primarily within `Proxy > Intercept` to manage in-flight HTTP transactions:

| Action / Capability | Windows / Linux | macOS | Operational Notes |
| :--- | :---: | :---: | :--- |
| **Forward Intercepted Request** | `Ctrl + F` | `⌘ + F` | Immediately dispatches the buffered request to the destination server. |
| **Drop Intercepted Request** | `Ctrl + D` | `⌘ + D` | Terminates the socket, preventing the request from reaching the server. |
| **Toggle Intercept On / Off** | `Ctrl + T` | `⌘ + T` | Global toggle switch to arm or disarm background interception. |

---

## ⚡ 3. Repeater Execution & Workspace Management

| Action / Capability | Windows / Linux | macOS | Operational Notes |
| :--- | :---: | :---: | :--- |
| **Send Request** | `Ctrl + Space` | `⌘ + Space` | Dispatches the active payload without lifting hands from the keyboard. |
| **Cancel Pending Request** | `Esc` or `Ctrl + C` | `Esc` | Aborts long-running requests or server timeouts. |
| **New Blank Request Tab** | `Ctrl + N` | `⌘ + N` | Instantiates a fresh raw request template. |
| **Duplicate Current Tab** | `Ctrl + Shift + D` | `⌘ + Shift + D` | Rapidly clones requests to populate Tab Groups for Race Condition testing. |
| **Undo / Redo Request History** | `Ctrl + [` / `Ctrl + ]` | `⌘ + [` / `⌘ + ]` | Navigates backward and forward across modification states within a tab. |

---

## 🔤 4. Inline Text Transformation & Quick Encoding

These hotkeys execute immediately upon highlighting any string within an editable HTTP request pane:

| Transformation Type | Windows / Linux | macOS | Transformation Output Example |
| :--- | :---: | :---: | :--- |
| **URL-Encode (Key Characters)** | `Ctrl + U` | `⌘ + U` | Converts delimiters: `admin 1` → `admin+1` or `%20` |
| **URL-Decode** | `Ctrl + Shift + U` | `⌘ + Shift + U` | Reverses hex encoding: `%27%20OR%201=1` → `' OR 1=1` |
| **HTML-Encode** | `Ctrl + H` | `⌘ + H` | Encodes injection markers: `<script>` → `&lt;script&gt;` |
| **HTML-Decode** | `Ctrl + Shift + H` | `⌘ + Shift + H` | Restores raw characters from HTML entities. |
| **Base64-Encode** | `Ctrl + B` | `⌘ + B` | Converts cleartext: `admin:admin` → `YWRtaW46YWRtaW4=` |
| **Base64-Decode** | `Ctrl + Shift + B` | `⌘ + Shift + B` | Restores raw cleartext from Base64 tokens. |

---

## 🔍 5. Editor Navigation & Search Operations

| Action / Capability | Windows / Linux | macOS |
| :--- | :---: | :---: |
| **Find in Request / Response** | `Ctrl + F` | `⌘ + F` |
| **Find Next Occurrence** | `Enter` or `F3` | `Enter` or `⌘ + G` |
| **Find Previous Occurrence** | `Shift + Enter` | `Shift + Enter` |
| **Go to Line Number** | `Ctrl + G` | `⌘ + L` |
| **Undo Last Edit** | `Ctrl + Z` | `⌘ + Z` |
| **Redo Edit** | `Ctrl + Y` | `⌘ + Y` |

---

## ⚙️ 6. Customizing Burp Suite Hotkeys

If default hotkeys conflict with your host operating system or tiling window manager:

1. Navigate to: **Settings > User interface > Hotkeys**.
2. Locate the target action in the searchable hotkey registry table.
3. Double-click the binding and press your desired physical key combination.
4. Click **Apply** to activate the new shortcut profile immediately.

> [!TIP]
> **Pro Tip:** Bind an unmapped global key (such as `F8` or `Pause/Break`) to `Proxy: Toggle intercept on/off`. This allows you to freeze and release requests effortlessly with one keystroke while navigating in your external browser!

---

| [← Previous: 19. Common Beginner Mistakes](./19-beginner-mistakes.md) | [TOC](../README.en.md#🗂️-table-of-contents-english-version---en) | [Next: Reference Materials & Resources →](./resources.md) |
