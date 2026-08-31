---
layout: page
title: Technical Advisory – Meta Horizon Shell — Automatic Dangerous Permission Grant via Virtual Input Injection
permalink: /vulns-without-cves/2026-meta-horizon-shell-201.0.0.649.547-permission-grant/
---

#### Original post: [https://djini.ai/technical-advisory-meta-horizon-shell-automatic-dangerous-permission-grant-via-virtual-input-injection/](https://djini.ai/technical-advisory-meta-horizon-shell-automatic-dangerous-permission-grant-via-virtual-input-injection/)

`By Ken Gannon — August 10, 2026`

|    **Product**    | Meta Horizon Shell (`com.oculus.vrshell`) |
| **Affected versions** | `201.0.0.649.547` |
| **Fixed in** | `204.0.0.558.431` — the virtual-input broadcast receivers are now registered with `RECEIVER_NOT_EXPORTED` |
| **Vulnerability type** | Unprotected exported activity + unauthenticated dynamic broadcast receivers on `VirtualInputDeviceManager` → arbitrary virtual key/mouse event injection → automatic acceptance of dangerous permission dialogs |
| **Attacker model** | Any third-party application installed on the device; zero user interaction required after install |
| **Tested on** | Meta Quest 3S |

## Note to the Reader

This vulnerability was identified during security research supported by Djini.ai's Deep Scan capabilities.

We also want to highlight that this exploit could have been used to enter Pwn2Own Ireland 2025. During that event, a valid entry against the Oculus 3S line of devices would have to demonstrate that a third party application could access the External Camera and Microphone hardware, without user consent.

<div align="center">
    <img src="/assets/2026-meta-horizon-shell-permission-grant/pwn2own-rules-2025.png" width="500">
    <p><em>Rules for Pwn2Own Ireland 2025 outlining the rules for Oculus 3/3S</em></p>
</div>

## Summary

It was found that `GauntletTestPanelActivity` in Meta Horizon Shell (`com.oculus.vrshell`) registers two dynamic broadcast receivers without a sender permission parameter, allowing any installed application to inject arbitrary key events and mouse clicks into the VR input system via the `VirtualInputDeviceManager` system service.

This enables a malicious application to automatically navigate and accept all Android dangerous permission dialogs — granting itself full access to the headset's passthrough cameras, microphone, and storage — without any user interaction.

## Impact

A malicious application installed on a Meta Quest 3S can exploit this issue to silently grant itself dangerous permissions (`CAMERA`, `RECORD_AUDIO`, `READ_MEDIA_IMAGES`, `READ_MEDIA_VIDEO`, `READ_MEDIA_AUDIO`) and immediately exercise those permissions to photograph the user's physical surroundings via the passthrough cameras, record ambient audio and conversations via the microphone, and read or write any media file on the device. The entire attack chain is fully automated and requires zero user interaction after the malicious application is installed. The permission grants persist across reboots.

## Exploit

To exploit this issue, the following PoC application must be installed on the target device. The application exploits the vulnerability automatically on launch — it starts the vulnerable `GauntletTestPanelActivity`, requests dangerous permissions, injects a 13-key sequence to navigate and accept all permission dialogs, then demonstrates access to the camera, microphone, and storage.

* The source code for the PoC APK can be found here: [https://github.com/mobilehackinglab/technical-advisory-files/tree/main/meta-horizon-vrshell-input-injection/poc-apk-source-code](https://github.com/mobilehackinglab/technical-advisory-files/tree/main/meta-horizon-vrshell-input-injection/poc-apk-source-code)
* The APK file itself can be found here: [https://github.com/mobilehackinglab/technical-advisory-files/raw/refs/heads/main/meta-horizon-vrshell-input-injection/com.test.vrshell_keyinject.apk](https://github.com/mobilehackinglab/technical-advisory-files/raw/refs/heads/main/meta-horizon-vrshell-input-injection/com.test.vrshell_keyinject.apk)

When the PoC application is launched, it performs the following steps automatically:

1. **Starts `GauntletTestPanelActivity`** (0s) — this registers the unprotected broadcast receivers for key injection.
2. **Requests dangerous permissions** (3s) — the Android permission dialog appears.
3. **Injects 13 key events** (6s–12s) — navigates and accepts all permission dialogs via `TRIGGER_VIRTUAL_KEYBOARD` broadcasts at 500ms intervals.
4. **Proves access** (14s) — captures a photo via the passthrough camera, records 5 seconds of microphone audio, writes a proof file to shared storage, then plays back the recorded audio through the headset speakers.

<div align="center">
    <img src="/assets/2026-meta-horizon-shell-permission-grant/vrshell-poc-demo.gif" width="500">
    <p><em>PoC APK running on an Oculus 3S device</em></p>
</div>

After the PoC completes, the following proof artifacts can be found on the device:

* `/sdcard/DCIM/poc_camera_proof.jpg` — photograph captured via the headset's passthrough camera
* `/sdcard/Music/poc_mic_proof.m4a` — microphone recording (AAC, 48kHz)
* `/sdcard/Documents/poc_storage_proof.txt` — storage write proof with timestamp

## Technical Details

#### Vulnerable Component: GauntletTestPanelActivity

`GauntletTestPanelActivity` is declared in the `com.oculus.vrshell` manifest without an `android:exported` attribute but with an intent-filter, and with `targetSdkVersion=29`. Per Android's auto-export behavior, this makes it exported and accessible to any application on the device without any permission.

#### Unprotected Dynamic Broadcast Receivers

In `onCreate()`, `GauntletTestPanelActivity` dynamically registers two broadcast receivers using the 2-argument `registerReceiver(receiver, filter)` variant, which does not specify a sender permission:

```java
IntentFilter keyboardFilter = new IntentFilter("com.oculus.gauntlet.TRIGGER_VIRTUAL_KEYBOARD");
registerReceiver(virtualKeyboardReceiver, keyboardFilter);

IntentFilter mouseFilter = new IntentFilter("com.oculus.gauntlet.TRIGGER_VIRTUAL_MOUSE");
registerReceiver(virtualMouseReceiver, mouseFilter);
```

Any application can send broadcasts matching these action strings, and the receivers will process them.

#### Key Event Injection via VirtualInputDeviceManager

When a `TRIGGER_VIRTUAL_KEYBOARD` broadcast is received, the receiver extracts the `keycode` integer extra and delegates to `GauntletTestPanelMainFragment.triggerVirtualKeyEvent(keycode)`:

```java
public void triggerVirtualKeyEvent(int keycode) {
    VirtualInputDeviceManager virtualInputDeviceManager0 =
        (VirtualInputDeviceManager) getContext()
            .getSystemService("virtual_input_device");

    if (virtualInputDeviceManager0 == null || this.virtualKeyboard != null) {
        Log.e("GauntletTestPanel",
            "Failed to get VirtualInputDeviceManager service");
    }

    if (this.virtualKeyboard == null) {
        this.virtualKeyboard = virtualInputDeviceManager0.createVirtualKeyboard(
            new VirtualKeyboardConfig.Builder()
                .setInputDeviceName("virtual-keyboard")
                .build());
    }

    this.virtualKeyboard.sendKeyEvent(
        new KeyEvent(0L, 0L, 0, keycode, 0));      // ACTION_DOWN
    // 100ms delay
    this.virtualKeyboard.sendKeyEvent(
        new KeyEvent(0L, 0L, 1, keycode, 0));      // ACTION_UP
}
```

The method obtains the `VirtualInputDeviceManager` system service, creates a `VirtualKeyboardExt` on the first call (cached in `this.virtualKeyboard` for subsequent calls), and sends a key-down followed by key-up event with the attacker-supplied keycode. There is no validation on the keycode value and no sender identity check.

The `com.oculus.vrshell` process holds the `android.permission.INJECT_EVENTS` system permission, which is required to create virtual input devices. The `:GauntletTest` process (where `GauntletTestPanelActivity` runs) inherits this permission, allowing the `VirtualInputDeviceManager` API calls to succeed.

The error log message "Failed to get VirtualInputDeviceManager service" is misleading — it fires on every call after the first because the condition `this.virtualKeyboard != null` is true (the keyboard was cached). The code falls through to `sendKeyEvent()` using the cached keyboard, which succeeds on every call.

#### Permission Dialog Navigation via Key Injection

When the PoC requests dangerous permissions, the Android `GrantPermissionsActivity` appears. The injected key events are delivered to whichever window has focus — in this case, the permission dialog. The 13-key sequence navigates the Quest 3S permission dialogs as follows:

| Keys | Action | Dialog |
|---|---|---|
| SPACE | Enable "headset cameras" checkbox | Camera permission |
| TAB x 4 | Navigate to next control | Camera permission |
| SPACE | Save camera preference | Camera permission |
| TAB x 3 | Navigate to "Allow" button | Camera permission |
| ENTER | Accept camera permission | Camera permission |
| SPACE | Accept microphone permission | Microphone permission |
| SPACE | Accept audio storage permission | Storage permission |
| SPACE | Accept images/video storage permission | Storage permission |

After the key sequence completes, all requested dangerous permissions are granted without the user touching any control.

## Recommendation / Remediation

This issue is fixed in Meta Horizon Shell `204.0.0.558.431` (vros 204). `GauntletTestPanelActivity` still exists and is still exported, but the two virtual-input broadcast receivers are now registered with the explicit `RECEIVER_NOT_EXPORTED` flag instead of the implicitly-exported 2-argument `registerReceiver`:

```
const/4 v3, 0x4                  # Context.RECEIVER_NOT_EXPORTED
invoke-virtual {p0, v0, v1, v3}, Landroid/content/Context;->registerReceiver(Landroid/content/BroadcastReceiver;Landroid/content/IntentFilter;I)Landroid/content/Intent;
```

In the vulnerable `201.0.0.649.547` build these used the 2-argument `registerReceiver(receiver, filter)` form, which is exported by default for the app's `targetSdkVersion=29`. The `RECEIVER_NOT_EXPORTED` flag restricts delivery to the app's own UID (and the system), so a third-party application can no longer deliver `com.oculus.gauntlet.TRIGGER_VIRTUAL_KEYBOARD` / `TRIGGER_VIRTUAL_MOUSE` to the receivers.

The action filters are built only in `GauntletTestPanelActivity`, there is no manifest-declared receiver for these actions, and the injection path (`VirtualInputDeviceManager.createVirtualKeyboard`/`sendKeyEvent`) is reachable only from the receiver's `onReceive` — so making the receivers non-exported closes the attack surface.

## Timeline

| Date | Summary |
|---|---|
| 2026-04-23 | Vulnerability discovered and PoC developed (vrshell 201.0.0.649.547) |
| 2026-04-23 | Vulnerability reported to vendor |
| 2026-06-16 | Vendor reported the issue patched; confirmed fixed in vrshell 204.0.0.558.431 — both receivers now registered `RECEIVER_NOT_EXPORTED`; third-party PoC no longer injects or auto-grants permissions |
