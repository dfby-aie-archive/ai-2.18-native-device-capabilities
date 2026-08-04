# Pre-Reading: Lesson 2.18: Integration of Native Device Capabilities in Cross-Platform Applications

Timebox **2–3 hours** across these resources before the lesson. You do not need to memorise everything; the goal is to build a mental model so the hands-on lab makes sense faster.

---

## 1. How Expo Handles Native Permissions

**Read (15 min)**

- [Expo: Permissions](https://docs.expo.dev/guides/permissions/): Read the full page. Focus on why permissions must be declared in `app.json` and how runtime permission requests work.

**Key ideas to take away:**

- Native permissions are a two-step process: declare the permission in `app.json` at build time, then request it from the user at runtime using an Expo SDK function.
- iOS and Android handle permissions differently. iOS requires a human-readable usage description string for each permission; Android grants permissions at the OS level without a custom message.
- Never assume a permission has been granted. Always check the result of a permission request before using the feature.

---

## 2. Camera and Media Library

**Read (20 min)**

- [Expo Camera docs](https://docs.expo.dev/versions/latest/sdk/camera/): Read the "Installation" section, the "Configuration in app.json" section, and the `CameraView` component reference. Note the `useCameraPermissions` hook, the `facing` prop, and `onBarcodeScanned`.
- [Expo Media Library docs](https://docs.expo.dev/versions/latest/sdk/media-library/): Read the "Installation" section and the `requestPermissionsAsync` and `createAssetAsync` function signatures.

**Key ideas to take away:**

- `useCameraPermissions` returns a permission object and a function to request permission. Check both that the permission object is not null and that it is granted before rendering `CameraView`.
- The camera permission (to show live preview and capture) and the media library permission (to save photos to the device) are separate. A user can grant one and deny the other.
- `CameraView`'s `onBarcodeScanned` prop fires on every frame where a barcode is detected. Your app must manage state to prevent acting on every repeated scan.

---

## 3. Image Picker

**Read (10 min)**

- [Expo Image Picker docs](https://docs.expo.dev/versions/latest/sdk/imagepicker/): Read the "Installation" section, the `launchImageLibraryAsync` function reference, and the result object structure. Note the `mediaTypes`, `allowsEditing`, `aspect`, and `quality` options.

**Key ideas to take away:**

- `launchImageLibraryAsync` triggers the OS permission dialog automatically on first use; you do not need to request the photo library permission manually before calling it.
- The result is an object with a `canceled` boolean and an `assets` array. Always check `canceled` before reading `assets[0].uri`.

---

## 4. Location

**Read (15 min)**

- [Expo Location docs](https://docs.expo.dev/versions/latest/sdk/location/): Read the "Installation" section, the `requestForegroundPermissionsAsync` function, and `getCurrentPositionAsync`. Note the structure of the returned location object (specifically `coords.latitude` and `coords.longitude`).
- [react-native-maps README](https://github.com/react-native-maps/react-native-maps): Read the "Installation" section and the `MapView` and `Marker` component props. Focus on `region`, `latitude`, `longitude`, `latitudeDelta`, and `longitudeDelta`.

**Key ideas to take away:**

- Location permission must be requested explicitly before calling `getCurrentPositionAsync`. The `requestForegroundPermissionsAsync` function returns a `status` field; check that it equals `"granted"` before proceeding.
- `MapView` requires a `region` object with four values: `latitude`, `longitude`, `latitudeDelta`, and `longitudeDelta`. The delta values control the zoom level.
- Location data is not always available instantly, especially on a physical device. Always use a loading state so the UI does not appear frozen.

---

## 5. Authentication Patterns and Biometrics

**Read (20 min)**

- [React Navigation: Authentication Flows](https://reactnavigation.org/docs/auth-flow): Read the full page. This is the pattern used in the lesson to conditionally render the auth navigator or the app navigator based on login state.
- [Expo Local Authentication docs](https://docs.expo.dev/versions/latest/sdk/local-authentication/): Read the function references for `hasHardwareAsync`, `isEnrolledAsync`, and `authenticateAsync`.

**Key ideas to take away:**

- The React Navigation authentication pattern uses a single `AuthContext` with a boolean `isAuthenticated` value. The navigator re-renders automatically when context changes, so there is no need to navigate programmatically after login.
- Before calling `authenticateAsync`, always check both `hasHardwareAsync` (does the device have biometric hardware) and `isEnrolledAsync` (has the user configured a fingerprint or face). Show the biometric option only when both checks pass.
- `authenticateAsync` returns an object with a `success` boolean. Only update your auth state if `result.success` is `true`.

---

## 6. Persisting Data with AsyncStorage

**Read (15 min)**

- [AsyncStorage: GitHub README](https://github.com/react-native-async-storage/async-storage#readme): Read the "Getting started" section and the `getItem`, `setItem`, and `removeItem` function references.

**Key ideas to take away:**

- AsyncStorage is a simple key-value store that persists data across app restarts. All operations are asynchronous and return Promises, so use `await` inside `async` functions.
- On app startup, call `getItem` inside a `useEffect` to restore any saved session before rendering the main UI. Use a `finally` block to ensure a loading state is cleared even if the read fails.
- AsyncStorage stores strings only. To store complex objects, use `JSON.stringify` before writing and `JSON.parse` after reading.

---

## Reflection (5 min)

Before the lesson, write down answers to these three questions:

1. In your own words, what is the difference between declaring a permission in `app.json` and requesting a permission at runtime?
2. Why would a user be able to deny the media library permission even after granting the camera permission?
3. What is one concept from the pre-reading that you would like the instructor to demonstrate more clearly?

Bring question 3 to class.
