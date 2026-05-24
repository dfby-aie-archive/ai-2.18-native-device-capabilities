# Assessment: Integration of Native Device Capabilities in Cross-Platform Applications

## Overview

- **Lesson:** Integration of Native Device Capabilities in Cross-Platform Applications / 2.18
- **Format:** 30 questions (MCQ and True/False)
- **Time:** ~30 minutes
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

---

### Q11

Why does the lesson check `Platform.OS === "ios"` before requesting media library permission inside `takePhoto`?

A - `MediaLibrary.requestPermissionsAsync` is only available on iOS

B - Android handles the media library permission at the OS level, so the runtime request is only necessary on iOS

C - Android always rejects the media library permission request and the call must be skipped

D - The `Platform.OS` check prevents the function from running on the Android emulator

---

### Q12

`CameraView`'s `onBarcodeScanned` prop fires a callback each time a barcode is detected in the camera frame. Why is this a problem without additional state management?

A - It causes `CameraView` to unmount after the first scan

B - The callback fires on every video frame where a barcode is visible, so a single barcode triggers dozens of navigations in rapid succession

C - `onBarcodeScanned` requires a unique barcode ID to prevent duplicates, which is not provided by the OS

D - It is not a problem; React Navigation deduplicates navigation calls automatically

---

### Q13

Which React Navigation hook is used to reset `isScanned` to `false` each time the Camera screen gains focus?

A - `useEffect` with an empty dependency array

B - `useScreenFocus`

C - `useFocusEffect`

D - `useNavigationState`

---

### Q14 (True/False)

`useFocusEffect` requires its callback to be wrapped in `useCallback` to prevent it from running on every render instead of only when the screen gains focus.

A - True

B - False

---

### Q15

A developer sets `onBarcodeScanned={isScanned ? undefined : handleBarcodeScanned}` on `CameraView`. What does passing `undefined` achieve?

A - It disables the camera entirely until `isScanned` is reset to `false`

B - It prevents `CameraView` from rendering while scanning is paused

C - It removes the barcode listener, stopping further scan callbacks until the prop is restored

D - It throws a prop-type warning but has no functional effect

---

### Q16

Which function from `expo-location` requests permission to access the device's location while the app is in use?

A - `Location.askPermissionAsync`

B - `Location.requestForegroundPermissionsAsync`

C - `Location.requestLocationAsync`

D - `Location.getPermissionsAsync`

---

### Q17

A developer calls `Location.getCurrentPositionAsync({})` and receives a location object. Which property contains the latitude and longitude?

A - `location.latitude` and `location.longitude`

B - `location.position.lat` and `location.position.lng`

C - `location.coords.latitude` and `location.coords.longitude`

D - `location.data.lat` and `location.data.lon`

---

### Q18

The lesson uses a `finally` block in the location fetch function. What is the purpose of `finally` here?

A - To catch errors that are not caught by the `try` block

B - To ensure `setRetrievingLocation(false)` is always called, whether the request succeeded or failed

C - To re-request the location permission if the first request was denied

D - To navigate back to the Home screen if the location fetch times out

---

### Q19

Which two props on `react-native-maps`'s `MapView` define the visible area of the map?

A - `zoom` and `center`

B - `region` and `initialRegion`

C - `latitude` and `longitude`

D - `bounds` and `viewport`

---

### Q20 (True/False)

`latitudeDelta` and `longitudeDelta` in a `MapView` `region` object control the zoom level: smaller values zoom in, and larger values zoom out.

A - True

B - False

---

### Q21

What is the purpose of the `NavigationApp` component being defined separately from the root `App` component in the lesson's `App.js`?

A - To allow `NavigationContainer` to be rendered outside the `AuthProvider` tree

B - To ensure `useContext(AuthContext)` is called inside the `AuthProvider` tree, so it reads the correct context value

C - To prevent the navigator from re-rendering when `AuthContext` changes

D - `NavigationApp` is a React Navigation requirement for authenticated flows

---

### Q22

Which `expo-local-authentication` function checks whether the device hardware supports biometric authentication?

A - `LocalAuthentication.isBiometricAvailable()`

B - `LocalAuthentication.checkBiometrics()`

C - `LocalAuthentication.hasHardwareAsync()`

D - `LocalAuthentication.getBiometricType()`

---

### Q23

A developer calls `LocalAuthentication.authenticateAsync()` on a device where the user has not enrolled any fingerprints or a face. What is the expected result?

A - The function prompts the user to enrol a fingerprint before continuing

B - The function returns `{ success: false }` or shows a system error, because there is no enrolled biometric to verify against

C - The function falls back to a PIN prompt automatically and returns `{ success: true }` if the PIN is correct

D - The function throws an unhandled exception that crashes the app

---

### Q24 (True/False)

`AsyncStorage.getItem` returns a string, or `null` if the key does not exist.

A - True

B - False

---

### Q25

A developer stores `"true"` in AsyncStorage using `AsyncStorage.setItem("isLoggedIn", "true")`. They later read the value and check `if (value === true)`. Why does this check fail?

A - `AsyncStorage.getItem` always returns `undefined` rather than `null` for missing keys

B - AsyncStorage stores and returns strings only; `value` is the string `"true"`, which is not strictly equal to the boolean `true`

C - `setItem` converts the value to an integer, so `value` is `1`, not `"true"`

D - The key `"isLoggedIn"` is reserved and cannot be used with `AsyncStorage`

---

### Q26

Why does the lesson initialise `isLoading` to `true` in `AuthContext` rather than `false`?

A - `true` is the default for all boolean state in React Native

B - The app must assume it is loading until `AsyncStorage.getItem` completes; starting at `false` would briefly render the wrong navigator before the check finishes

C - AsyncStorage requires `isLoading` to be `true` before `getItem` can be called

D - `NavigationContainer` will not render until `isLoading` is `true`

---

### Q27

What is rendered in `NavigationApp` while `isLoading` is `true`?

A - The Login screen with a disabled form

B - A blank white screen

C - An `ActivityIndicator` centred on the screen

D - The app's splash screen

---

### Q28

A developer removes the `isLoading` guard from `NavigationApp`. What visible problem occurs for users who are already logged in?

A - The AsyncStorage read fails silently and the user is always logged out

B - The Login screen flashes briefly on every app launch before the session is restored and the app navigator appears

C - The tab navigator renders with no screens registered

D - `NavigationContainer` throws an error because the navigator tree is empty

---

### Q29

Which package should be used instead of `AsyncStorage` when storing sensitive data such as authentication tokens in a production app?

A - `expo-file-system`

B - `expo-sqlite`

C - `expo-secure-store`

D - `@react-native-community/encrypted-storage`

---

### Q30 (True/False)

In the lesson's auth pattern, calling `logout` navigates the user to the Login screen by calling `navigation.navigate("Login")`.

A - True

B - False

---
