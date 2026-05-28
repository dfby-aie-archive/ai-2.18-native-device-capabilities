# Assessment: Integration of Native Device Capabilities in Cross-Platform Applications

## Overview

- **Lesson:** Integration of Native Device Capabilities in Cross-Platform Applications / 2.18
- **Format:** 10 questions (MCQ and True/False)
- **Time:** ~10–15 minutes
- **Scoring:** 1 point each

## Questions

### Q1

Where in an Expo project do you declare which native capabilities (such as camera access or location) your app intends to use?

A - In `package.json` under a `permissions` key

B - In `app.json` using the `plugins` and `infoPlist` / `androidPermissions` options

C - In a separate `permissions.js` file at the project root

D - In each screen component by calling `Permissions.declare()` on mount

---

### Q2 (True/False)

iOS requires a human-readable usage description string for each permission, while Android grants runtime permissions without needing a custom description string in the app config.

A - True

B - False

---

### Q3

A developer calls `expo-image-picker`'s `launchImageLibraryAsync` without requesting any permission first. What happens on first use?

A - The function throws a `PermissionDeniedException`

B - The function returns `{ canceled: true }` without opening the picker

C - The OS triggers the photo library permission dialog automatically

D - The function silently succeeds but returns an empty `assets` array

---

### Q4

Which of the following correctly reads the URI of the first image selected from `launchImageLibraryAsync`?

A - `result.uri`

B - `result.assets.uri`

C - `result.assets[0].uri`

D - `result.image.uri`

---

### Q5

A developer checks `result.canceled` before reading the selected image URI. Why is this check necessary?

A - `result.canceled` being `true` throws an error if `result.assets` is accessed

B - The user may have closed the picker without selecting anything, leaving `result.assets` undefined or empty

C - `result.canceled` must be `false` before the media library permission is considered granted

D - The check is not necessary; `result.assets[0]` is always defined even if the user cancelled

---

### Q6

Which hook from `expo-camera` is used to request and track camera permission status?

A - `usePermission`

B - `useCameraPermissions`

C - `useCameraAccess`

D - `useMediaPermissions`

---

### Q7

`useCameraPermissions` returns an array of two values. What are they?

A - `[permissionStatus, errorMessage]`

B - `[hasPermission, grantPermission]`

C - `[permission, requestPermission]`

D - `[isGranted, isDenied]`

---

### Q8

A developer renders `CameraView` directly without checking the permission object. What is the most likely result?

A - `CameraView` requests the permission itself and shows a native dialog

B - The app crashes or shows a blank screen because the camera is not available until permission is granted

C - `CameraView` renders in a "preview-only" mode with no permission required

D - React Native queues the render and retries once permission is granted automatically

---

### Q9 (True/False)

The camera permission and the media library permission are the same permission. Granting one automatically grants the other.

A - True

B - False

---

### Q10

On iOS, a developer calls `MediaLibrary.createAssetAsync(photo.uri)` without first requesting media library permission. What is the most likely outcome?

A - The photo is saved successfully because the camera permission covers all media operations

B - The function throws a visible error that is caught by React Native's error boundary

C - The function silently fails or throws, because the media library permission is a separate, independent permission

D - The OS shows the permission dialog automatically and proceeds if the user grants it
