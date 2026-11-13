# SmartBin

SmartBin is an Expo app for monitoring the ESP32 sensor at the Firebase Realtime Database path `/smartbin`. It uses Expo Router, React Native, TypeScript, and the Firebase JavaScript SDK. Firebase Analytics is not used.

## Open and run the project

1. In VS Code, open the `SmartBin` folder.
2. Open **Terminal → New Terminal**. The terminal must be inside the `SmartBin` folder.
3. Start Expo:

   ```powershell
   npx.cmd expo start
   ```

4. Open Expo Go on your Android phone and scan the QR code. Your computer and phone should be on the same Wi-Fi network.

If PowerShell says that `npx.ps1` cannot run, use `npx.cmd` as shown above. This avoids changing your computer's script policy.

## Firebase setup

The app's Firebase settings are read by [`lib/firebase.ts`](./lib/firebase.ts) from environment variables. Do not paste private keys or service-account JSON into the app.

1. In the Firebase Console, open **Project settings** (the gear icon) → **General** → **Your apps**. Select your web app, or add one if needed, then choose **SDK setup and configuration → Config**.
2. Copy these values into the matching lines in `.env` in the `SmartBin` folder:
   - `apiKey`, `authDomain`, `projectId`, and `appId`: the web app's Firebase config.
   - `databaseURL`: in **Build → Realtime Database**, open **Data** and copy the database URL shown above the data tree.
3. Enable **Authentication → Sign-in method → Anonymous**. The app signs in automatically; you do not need to create an email/password account.
4. In **Realtime Database → Rules**, publish the rules from [`database.rules.json`](./database.rules.json). They allow signed-in sessions to read the database and write `/smartbin`.
5. Stop Expo, then restart it so the updated `.env` values load:

   ```powershell
   npx.cmd expo start --clear
   ```

Firebase web-app config values identify your Firebase project; they are not service-account credentials. Do not use the ESP32 password as an app setting.

Anonymous authentication avoids manually creating accounts, but it is not device-specific access control. The included rules allow any authenticated session to write `/smartbin`. For a private deployment, restrict writes to the ESP32's own identity or put a trusted server between the device and database.

The ESP32 must also authenticate before writing. Enable **Anonymous** sign-in in Firebase Authentication, then have the firmware call Firebase Authentication's anonymous sign-up endpoint with the project's Web API key and use the returned ID token when writing to Realtime Database. Do not put a service-account key in firmware. Anonymous identities can be cleared if the ESP32's auth state is erased, so the firmware should sign in again when its token expires.

### Data expected by the app

The ESP32 should update this exact path and these field names:

```json
{
  "smartbin": {
    "fillLevel": 65,
    "distanceCm": 12.3,
    "binState": "yellow",
    "esp32Online": true,
    "lastSeen": 1763020000000
  }
}
```

The app uses `fillLevel`, `distanceCm`, `binState`, and `lastSeen`. It deliberately ignores `esp32Online`: Online/Offline is calculated from `lastSeen` and Firebase's `.info/serverTimeOffset`, and the ESP32 is considered offline when its last update is more than 15 seconds old.

## Lid control

The **Lid control (coming soon)** card is disabled. This project has no servo or lid control yet.

## Troubleshooting

- **Expo Go cannot load the app:** confirm that the phone and computer use the same Wi-Fi, then restart with `npx.cmd expo start --clear`. If school Wi-Fi blocks device connections, use Expo's tunnel connection option.
- **Red error screen:** read the first error line. Confirm the Expo SDK dependencies are installed, then restart Expo with `npx.cmd expo start --clear`. Share the complete error text when asking for help.
- **Permission denied:** confirm Anonymous sign-in is enabled, the database rules are published, and the ESP32 is sending a valid Firebase ID token with its database write.
- **No data yet:** confirm that the ESP32 is writing to `/smartbin` with the exact field names and correct data types shown above.
- **Offline:** the app uses `lastSeen`, not `esp32Online`. Confirm that the ESP32 updates `lastSeen` every five seconds using a Firebase server timestamp in milliseconds.

## Useful source files

- [`lib/firebase.ts`](./lib/firebase.ts): Firebase initialization and configuration.
- [`src/hooks/use-smart-bin.tsx`](./src/hooks/use-smart-bin.tsx): live database listeners and online timing.
- [`src/types/smartbin.ts`](./src/types/smartbin.ts): the sensor data type.
- [`src/components/FillGauge.tsx`](./src/components/FillGauge.tsx): fill-level gauge.
- [`src/components/StatusBadge.tsx`](./src/components/StatusBadge.tsx): bin and connection labels.
- [`src/app/index.tsx`](./src/app/index.tsx): dashboard screen.
- [`src/components/firebase-auth-gate.tsx`](./src/components/firebase-auth-gate.tsx): automatic anonymous sign-in and auth gate.

## Checks

Run these commands from the `SmartBin` folder:

```powershell
npx.cmd tsc --noEmit
npx.cmd eslint src/app/index.tsx src/app/settings.tsx src/hooks/use-smart-bin.tsx src/components/firebase-auth-gate.tsx src/components/FillGauge.tsx src/components/StatusBadge.tsx
```
