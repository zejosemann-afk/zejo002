# Local Bridge (Rino project)

A simple, offline-first companion between an **Android phone** and a **Windows PC**: copy and paste, file transfer, notifications, remote lock and unlock, and later a personal assistant that can act across both devices with permissions you grant one by one.

**Status:** Planning. Nothing is built yet.

---

## Goals

- **Simple:** copy text and send files in one tap. Few screens, few settings.
- **Offline-first:** works over the same Wi-Fi or the phone's hotspot. No internet and no account needed for the core features.
- **Private and secure:** encrypted end to end, paired by QR code.
- **Assistant (later):** it can lock the PC, send files and open apps, but only what you have allowed.

## Non-goals (for now)

- No cloud account or login.
- No iPhone, Mac or Linux support.
- No assistant with blanket access to everything on every device.

---

## How it works

```
 Android app (Kotlin)  <--- encrypted WebSocket, local network --->  Windows tray app (C#/.NET)
        |                                                                   |
   Keystore keys                                                      DPAPI-protected keys
        |                                                                   |
  (later) small AI model  <------------- or calls the PC's model ----------+
```

- **Pairing:** the Windows app shows a QR code, the phone scans it, and the two devices exchange keys.
- **Transport:** WebSockets over the local network with encryption. Bluetooth may be added later for discovery.
- **Relay (later, optional):** an end-to-end encrypted relay for when the devices are on different networks.

---

## Features and known platform limits

| Feature | Plan | Limit to design around |
|---|---|---|
| Phone to PC copy | One tap: share sheet, Quick Settings tile, or a keyboard key | Android 10+ blocks clipboard reads by background apps unless the app has focus or is the keyboard |
| PC to phone copy | Automatic | None known |
| File transfer | Over Wi-Fi or hotspot | Android file access needs user-chosen folders |
| Notifications | Mirror notifications to the PC | Google Play restricts SMS permissions to the default SMS, Phone or Assistant app, so mirror notifications instead of reading SMS |
| Remote lock | Windows lock command | None known |
| Unlock | Windows credential provider (C++) plus phone approval | See security below |
| Assistant | Model outputs JSON commands from a fixed list | Gemini Nano runs only on a short list of flagship phones, so run a small open model on the PC and let the phone call it |

---

## Security principles

1. **Unlock is never passive.** Bluetooth proximity unlock has been defeated by relay attacks, even with encryption. Unlock always needs an explicit tap plus fingerprint on the phone.
2. **Signed approvals.** The PC sends a one-time challenge, and the phone signs it after biometric approval.
3. **Credentials stay on the PC,** encrypted with Windows DPAPI, and are released only after a verified approval.
4. **Assistant least privilege.** A fixed list of allowed actions, confirmation for risky ones (unlock, delete, payments), and a log of everything it does.
5. **No secrets in this repository.** Never commit keys, tokens or passwords.

---

## Roadmap

### Phase 0: Setup
- Create the repository, choose a license, pick the project name.
- Write down the pairing and message format.
- **Done when:** the plan is written and a "hello" message goes from phone to PC on the same Wi-Fi.

### Phase 1: MVP (copy-paste and files)
- QR pairing with encrypted connection.
- One-tap phone to PC copy; PC to phone copy.
- Send files both ways.
- **Done when:** two devices pair in under a minute and copy text and a file with no internet.

### Phase 2: Notifications and lock
- Mirror phone notifications to the PC.
- Lock the PC from the phone; show phone battery on the PC.
- **Done when:** a notification appears on the PC and one tap locks it.

### Phase 3: Unlock (hardest)
- Write a threat model first.
- Build a Windows credential provider (C++) and the phone approval flow.
- Study existing open-source code before writing anything from scratch.
- **Done when:** the PC unlocks only after a tap and fingerprint on the phone, and a test shows nearby-phone-only does nothing.

### Phase 4: Assistant
- A small set of actions: lock PC, send a file, open an app.
- Small open model on the PC, usable offline; cloud model optional when online.
- Confirmations and an action log.
- **Done when:** the assistant completes those actions and refuses anything not on the list.

### Phase 5: Different networks (optional)
- End-to-end encrypted relay so the devices can connect away from the same Wi-Fi.

---

## Tech stack (proposed)

| Part | Choice |
|---|---|
| Android app | Kotlin and Jetpack Compose |
| Windows app | C# and .NET (tray app); C++ for the credential provider |
| Communication | WebSockets over the local network, with encryption (libsodium) |
| Key storage | Android Keystore; Windows DPAPI |
| Offline AI | llama.cpp or Ollama on the PC; small models only on the phone |

## Prior art to study

- Microsoft Phone Link and Link to Windows
- KDE Connect
- LocalSend
- Remote Fingerprint Unlock (Android app plus Windows credential provider)

## Open questions

- Project name
- License (for example MIT)
- Minimum versions (proposed: Android 10+, Windows 10/11)

## License

To be decided.
