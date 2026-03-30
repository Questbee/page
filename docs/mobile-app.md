# Mobile App Guide

---

## Overview

The Questbee mobile app is an offline-first data collection tool for field workers. It runs on Android and iOS, works with no internet connection, and automatically syncs collected data to your Questbee server when back online.

The app is distributed as a **prebuilt binary**:
- **Android:** APK download from the GitHub releases page
- **iOS (preview):** Via Expo Go during development

White-label compilation (your organization's name, logo, and colors) is available on **Professional and above**.

---

## Installation

### Android

1. Download the latest APK from the releases page on GitHub
2. On your Android device go to **Settings → Security → Unknown Sources** and allow installation from unknown sources
3. Open the downloaded `.apk` file and tap **Install**
4. Open **Questbee** from the app drawer

### iOS (Preview / Development)

1. Install the **Expo Go** app from the App Store
2. Get the Expo development URL from your Questbee administrator
3. Open Expo Go and scan or enter the URL

> **Note:** The iOS preview via Expo Go is for testing and piloting only. A compiled standalone iOS app is available on paid plans.

---

## Pairing with Your Server

Before collecting data, the app must be connected to your organization's Questbee server. Pairing is done by scanning a QR code — no separate username or password is required on the device. The QR code embeds your user identity, so the app is ready to collect immediately after scanning.

### Step 1 — Generate a Pairing QR Code (Dashboard)

1. Log into the Questbee web dashboard
2. Go to **Settings**
3. Under **Mobile Pairing**, click **Generate QR Code**
4. A QR code appears — it is valid for a short time (shown on screen)

> **Important:** Each QR code is single-use. If pairing fails or you need to pair again after unpairing, cancel the existing QR and generate a new one before scanning.

### Step 2 — Connect the App

1. Open the Questbee app on your phone
2. Tap **Scan QR Code**
3. Point the camera at the QR code on the dashboard screen

The app will:
- Verify the connection to your server
- Store your device token securely
- Download your assigned forms automatically

You are now ready to collect data. The welcome screen shows the server you are connected to and your assigned forms.

---

## Managing Connected Devices

Administrators and managers can see all paired devices in the dashboard under **Settings → Connected Devices**. Each entry shows the device label, the user it is linked to, and the last sync time.

To disconnect a device, click **Revoke** next to it. The device will no longer be able to sync until it is re-paired with a new QR code.

---

## Downloading Forms

Forms are downloaded automatically after pairing. To refresh manually:

1. Go to the **Forms** tab in the app
2. Pull down to refresh

Forms are stored on the device. Once downloaded, they are available with no internet connection.

> **Versioning:** When an administrator publishes a new version of a form, the app downloads it automatically on next sync. Submissions always record which version they were collected against.

---

## Collecting Data

### Start a Submission

1. Open the **Forms** tab
2. Tap the form you want to fill out
3. Fill in each field — swipe or tap **Next** to advance
4. Tap **Submit** when complete

### Field Types

| Field | How to Use |
|---|---|
| **Text / Long text** | Type your response |
| **Number** | Enter a number using the numeric keypad |
| **Email / Phone** | Type using the appropriate keyboard |
| **Date / Time** | Tap to open a date or time picker |
| **Single choice** | Tap one option |
| **Multiple choice** | Tap all options that apply |
| **GPS location** | Tap **Capture** to record your current position. Wait for accuracy to improve before confirming. |
| **GPS trace** | Tap **Start** to begin recording your path. Tap **Stop** when done. |
| **Photo** | Tap **Take Photo** to use the camera, or **Choose from Gallery** |
| **Audio** | Tap **Record** — tap again to stop |
| **Signature** | Draw with your finger on the signature pad |
| **Barcode / QR** | Tap **Scan** and point the camera at the code |
| **File attachment** | Tap to select a file from your device |
| **Repeating group** | Tap **Add entry** to fill in a set of fields multiple times (e.g., one per household member) |

### Conditional Questions

Some questions only appear based on your previous answers. The form shows only what is relevant to your responses.

### Required Fields

Fields marked with `*` must be answered before you can submit. If you tap Submit with missing required fields, the form scrolls to the first unanswered required field.

---

## Offline Mode

The app is designed to work without internet. When offline:

- You can fill out and submit forms normally
- Submissions are saved to the device queue
- Photos, audio, and other media are stored on the device
- GPS capture works using the device's GPS chip — no internet needed

The status indicator on the home screen shows:
- **Synced** — all submissions have been uploaded
- **X pending** — number of submissions waiting to sync
- **Offline** — device has no internet connection

---

## Syncing

Sync happens automatically when the device has a network connection. You do not need to do anything manually.

### What Gets Synced

- Submission data (all field responses)
- Photos, audio recordings, signatures, and file attachments
- GPS traces

Media files may take longer to upload than form data on slow connections. The app continues uploading in the background.

### Reliability

Each submission is sent with a unique device-generated ID. If the connection drops mid-sync, the app retries automatically. The server ignores duplicate submissions — you will never get duplicate records even if the same submission is sent twice.

---

## Multiple Devices

Multiple field workers can use different devices on the same Questbee server. Each device is paired independently and receives its own device token.

Submissions from all devices appear together in the dashboard, tagged with the device ID and the linked user account.

---

## Troubleshooting

**Can't connect after scanning QR code**
- Make sure your phone can reach the server (same Wi-Fi, or the server is publicly accessible)
- Ask your administrator to verify the server is running: `docker compose ps`
- The QR code expires after a short time — ask for a new one if it has been more than a few minutes
- **Each QR code can only be used once.** If you unpaired the device and are trying to pair again, cancel the current QR code in the dashboard and generate a fresh one before scanning

**Forms not appearing after pairing**
- Pull down to refresh on the Forms tab
- Check that your user account has published forms assigned to it

**Submissions stuck as pending**
- Check your network connection
- Open the app with a working connection — sync runs automatically
- If it continues, check with your administrator that the server is reachable

**GPS not capturing**
- Make sure location permission is granted: **Phone Settings → Apps → Questbee → Permissions → Location → Allow all the time**
- Move to an open area — GPS is unreliable indoors and under heavy tree cover
- Wait 30–60 seconds for accuracy to improve before confirming

**Photo / camera not working**
- Make sure camera permission is granted: **Phone Settings → Apps → Questbee → Permissions → Camera → Allow**

**App crashed or won't open**
- Force-stop the app and reopen it
- Unsynced submissions are saved in the device database — they will not be lost unless you clear app storage

---

## See Also

- [Getting Started](getting-started.md) — Server setup and first form
- [API Reference](api-reference.md) — Technical integration reference
- [Deployment Guide](deployment.md) — Self-hosting and production setup
