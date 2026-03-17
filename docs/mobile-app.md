# Mobile App Guide

> ⚠️ **Pre-MVP Notice:** This documentation describes the intended mobile app architecture and features. The product is in pre-MVP development and details are subject to change. Join the [pilot program](mailto:partnerships@questbee.io) to help shape the first release.

---

## Overview

The Questbee mobile app is a cross-platform (iOS + Android) offline-first data collection app built with **React Native / Expo**. It connects to your self-hosted Questbee server and allows field workers to collect data without any internet connectivity.

Key capabilities:
- **Offline-first**: All data is stored locally in SQLite (WatermelonDB) and synced when connectivity is available
- **QR code pairing**: Connect to your server by scanning a QR code — no manual URL entry
- **15+ field types**: Text, number, GPS, photo, barcode, signature, audio, select, date, and more
- **Conditional logic**: Show/hide fields based on previous answers
- **White-label ready**: Rebrand the app with your organization's name, logo, and colors

---

## Requirements

| Requirement | Minimum |
|---|---|
| iOS | 14.0 or later |
| Android | 8.0 (API 26) or later |
| Questbee server | Running and accessible on local network or internet |
| Storage | 200 MB free space |

---

## Installation

### Community Edition (Sideload / Expo Go)

For development and pilot use, the mobile app can be run using Expo Go:

1. Install [Expo Go](https://expo.dev/go) on your iOS or Android device.
2. Ensure your Questbee server is running:
   ```bash
   docker-compose up --build
   ```
3. From the Questbee project directory, start the Expo development server:
   ```bash
   cd mobile
   npx expo start
   ```
4. Scan the QR code shown in the terminal with your device's camera (iOS) or the Expo Go app (Android).

### Enterprise Edition (Compiled App)

For production deployments, the app is compiled into a standalone `.ipa` (iOS) or `.apk`/`.aab` (Android) file using EAS Build:

```bash
# Install EAS CLI
npm install -g eas-cli

# Log in to your Expo account
eas login

# Build for Android
eas build --platform android --profile production

# Build for iOS
eas build --platform ios --profile production
```

The compiled app can be distributed via:
- Internal app distribution (MDM / direct install)
- Google Play Store (internal track)
- Apple App Store (TestFlight for pilots)

> **Professional and above**: White-label Android APK compilation (custom bundle ID, app name, assets) is included in the Professional license. Full App Store + Play Store publishing (both iOS and Android) requires the Enterprise license or above, as it involves signing under the client's developer accounts.

---

## Connecting to Your Server

### QR Code Pairing (Recommended)

1. Open the web dashboard at `http://your-server:3000`.
2. Navigate to **Mobile → Pair Device**.
3. A QR code is displayed containing your server URL and a one-time pairing token.
4. Open the Questbee mobile app → tap **Pair with Server**.
5. Scan the QR code. The app connects and downloads your project's forms automatically.

### Manual Configuration

If QR pairing is unavailable:
1. Open the app → **Settings → Server Configuration**.
2. Enter your server URL (e.g., `https://questbee.yourorg.com` or `http://192.168.1.100:8000`).
3. Enter your login credentials or API key.
4. Tap **Connect**.

---

## Collecting Data

### Selecting a Form

1. On the home screen, tap the project you want to work in.
2. Tap a form to start a new submission.
3. The form loads from local storage — no internet required after initial sync.

### Field Types

| Field Type | Description |
|---|---|
| **Text** | Short or long text input |
| **Number** | Numeric input with optional range validation |
| **Select / Multi-select** | Dropdown or checkbox list of options |
| **GPS / Geopoint** | Records latitude, longitude, and altitude |
| **Photo** | Camera capture or gallery upload (multiple allowed) |
| **Barcode / QR** | Scan barcodes and QR codes via device camera |
| **Signature** | Touch-based signature pad |
| **Date / Time** | Date picker, time picker, or datetime |
| **Audio** | Voice recording |
| **File** | Document or file attachment |
| **Calculated** | Auto-computed value based on formula |
| **Note** | Read-only instructional text |
| **Repeat Group** | Repeatable section (e.g., household members) |
| **Rating** | Star or numeric rating |
| **Matrix** | Grid of questions with shared answer options |

### Conditional Logic

Fields can be shown or hidden based on the value of previous answers. Logic is configured in the form builder on the web dashboard and applied automatically in the mobile app — no extra setup required.

---

## Offline Sync

### How It Works

1. The app stores all submissions locally in a SQLite database (WatermelonDB).
2. When the app detects network connectivity (WiFi or cellular), it automatically starts a sync with the Questbee server.
3. Conflicts (e.g., same record edited on two devices) are resolved using a last-write-wins strategy with server timestamp authority.
4. Sync status is visible in the app header:
   - 🟢 **Synced** — all local data has been uploaded
   - 🟡 **Pending** — submissions queued for upload
   - 🔴 **Offline** — no connection detected

### Manual Sync

To force a sync:
1. Pull down on the home screen to trigger a manual sync.
2. Or navigate to **Settings → Sync → Sync Now**.

### Sync Conflicts

If a conflict is detected:
- The server's version is shown alongside the device's version.
- The user is prompted to choose which version to keep (or keep both).
- Conflict resolution can be configured to be automatic (server wins) in the admin settings.

---

## White-Label Configuration (Enterprise)

To white-label the app:

1. Update `mobile/app.config.js`:
   ```js
   export default {
     name: "Your App Name",
     slug: "your-app-slug",
     icon: "./assets/your-icon.png",
     splash: { image: "./assets/your-splash.png", backgroundColor: "#ffffff" },
     android: { package: "com.yourorg.yourapp" },
     ios: { bundleIdentifier: "com.yourorg.yourapp" },
   };
   ```
2. Replace assets in `mobile/assets/`:
   - `icon.png` (1024×1024)
   - `splash.png` (1284×2778)
   - `adaptive-icon.png` (Android, 1024×1024)
3. Update the primary color in `mobile/constants/theme.js`.
4. Rebuild using EAS Build (see Installation above).

---

## Troubleshooting

### App cannot connect to server
- Confirm the server is running: `docker-compose ps`
- Ensure your device is on the same network as the server (for local deployments)
- Check firewall rules — port 8000 (API) must be accessible
- Verify the server URL does not have a trailing slash

### Sync is stuck / not completing
- Check server logs: `docker-compose logs api`
- Ensure the device has sufficient storage space
- Try a manual sync via **Settings → Sync → Sync Now**
- If the problem persists, re-pair the device using a new QR code

### Forms are not appearing
- Ensure forms are published (not in draft) in the web dashboard
- Pull down to refresh the form list on the home screen
- Re-pair the device if the problem persists

### Photos are not syncing
- Large photo files may take longer — keep the app open until sync completes
- Check available storage on the server (see Deployment Guide)
- Configure max photo resolution in **Web Dashboard → Settings → Mobile → Photo Quality**

---

## Support

- **Community:** [GitHub Issues](https://github.com/questbee-io/questbee/issues)
- **Enterprise:** [hello@questbee.io](mailto:hello@questbee.io)
- **Partnerships:** [partnerships@questbee.io](mailto:partnerships@questbee.io)
