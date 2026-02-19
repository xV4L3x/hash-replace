<p align="center">
  <h1 align="center">🔐 HashReplace</h1>
  <p align="center">
    <b>Instantly SHA-1 hash any selected text — right from the iOS callout bar.</b>
  </p>
  <p align="center">
    <img src="https://img.shields.io/badge/platform-iOS-blue?style=flat-square" />
    <img src="https://img.shields.io/badge/architecture-iphoneos--arm-green?style=flat-square" />
    <img src="https://img.shields.io/badge/requires-MobileSubstrate-critical?style=flat-square" />
    <img src="https://img.shields.io/badge/version-0.0.1-orange?style=flat-square" />
  </p>
</p>

---

## ✨ What Is HashReplace?

**HashReplace** is a lightweight jailbreak tweak that adds a **"Sha1"** button directly into the native iOS text-selection callout bar (the bubble menu that appears when you select text). Tap it, and your selected text is instantly replaced with its **SHA-1 cryptographic hash** — no apps to open, no copy-pasting into a terminal.

> Select → Tap **Sha1** → Done. Your text is now a 40-character hex digest.

### Example

| Before | After |
|---|---|
| `hello world` | `2aae6c35c94fcfb415dbe95f408b9ce91ee846ed` |

---

## 🚀 Features

- **⚡ One-tap hashing** — SHA-1 hash computation embedded directly in the text selection menu
- **🔄 Non-destructive clipboard** — Automatically saves and restores your clipboard contents after the operation
- **📱 System-wide** — Works in every app that uses `UIKit` text selection (Safari, Notes, Messages, third-party apps, etc.)
- **🪶 Ultra-lightweight** — Single-file tweak with zero dependencies beyond MobileSubstrate
- **🧠 Smart loading** — Intelligently filters which processes it injects into, skipping system services like `AdSheet`, `CoreAuthUI`, `InCallService`, and app extensions

---

## 🪝 Hooks — Under the Hood

HashReplace uses **Logos/Theos** to hook into two key UIKit classes via **MobileSubstrate (Cydia Substrate)**:

### 1. `UICalloutBar`

| Hook | What It Does |
|---|---|
| `initWithFrame:` | Creates a custom `UIMenuItem` titled **"Sha1"** and attaches it to the callout bar instance on initialization. |
| `updateAvailableButtons` | Conditionally injects the "Sha1" button into the callout bar's `extraItems` array. The button only appears when the **cut** action is available (i.e., editable, selected text exists), ensuring it doesn't show up in read-only contexts. Uses `MSHookIvar` to read the private `m_currentSystemButtons` ivar. |

### 2. `UIResponder`

| Hook | What It Does |
|---|---|
| `slcOwO:` *(new method)* | Injected via `%new` — this is the action handler for the "Sha1" menu item. It performs a **cut → hash → paste** pipeline using `dispatch_after` to respect the async nature of `UIPasteboard`, and carefully restores the original clipboard contents afterward. |

### Constructor (`%ctor`)

The constructor implements a **smart process filter** to prevent the tweak from loading in inappropriate contexts:

- ✅ Loads in: Standard user-facing applications (`/Application/` or `/Applications/`)
- ❌ Skips: `AdSheet`, `CoreAuthUI`, `InCallService`, `MessagesNotificationViewService`, app extensions (`.appex`), and FileProvider processes

---

## 🔧 How It Works

```
┌──────────────────────────────────────────────────────┐
│  1. User selects text in any app                     │
│  2. Callout bar appears with native + "Sha1" button  │
│  3. User taps "Sha1"                                 │
│  ┌────────────────────────────────────────────────┐   │
│  │  a. Save current clipboard                     │   │
│  │  b. Programmatically trigger cut:              │   │
│  │  c. Read cut text from clipboard               │   │
│  │  d. Compute SHA-1 via CommonCrypto (CC_SHA1)   │   │
│  │  e. Write hex digest to clipboard              │   │
│  │  f. Programmatically trigger paste:            │   │
│  │  g. Restore original clipboard contents        │   │
│  └────────────────────────────────────────────────┘   │
│  4. Selected text is now replaced with its SHA-1 hash │
└──────────────────────────────────────────────────────┘
```

The hashing itself uses Apple's **CommonCrypto** `CC_SHA1()` function — the same battle-tested implementation used across macOS and iOS — producing a standard 40-character lowercase hexadecimal digest.

---

## 🏗️ Building

### Prerequisites

- A macOS machine with [Theos](https://theos.dev) installed
- An iOS SDK configured in Theos
- A jailbroken iOS device (or a compatible simulator setup)

### Build & Install

```bash
# Clone the repository
git clone https://github.com/xV4L3x/hash-replace.git
cd hash-replace

# Build the tweak
make

# Build the .deb package
make package

# Install to a connected device (via SSH)
make install
```

> **Note:** After installation, SpringBoard will automatically respring to activate the tweak.

---

## 📜 License

This project is provided as-is for educational and personal use on jailbroken iOS devices.
