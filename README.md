# 2.18 Integration of Native Device Capabilities in Cross-Platform Applications

## Lesson Overview

This lesson teaches learners how to access native device hardware and OS features in a React Native app built with Expo. Starting from a blank Expo project, learners build Explorer App: a four-tab app with a gallery image picker, a live camera view with photo capture and QR code scanning, a GPS location screen with a map, and a complete authentication flow including biometric login and persistent sessions backed by AsyncStorage.

## Dependencies

- [Self Studies](./studies.md)
- [Lesson](./lesson.md)
- [Assignment](./assignment.md)

## Lesson Objectives

- Configure iOS and Android permissions for native device features using Expo's `app.json` plugin system
- Integrate `expo-camera` and `expo-media-library` to build a camera screen with live preview, photo capture, and QR code scanning
- Use `expo-image-picker` to let users select media from the device library
- Implement location-based features using `expo-location` and display coordinates on an interactive map with `react-native-maps`
- Build an authenticated navigation shell with `AuthContext`, conditional navigator rendering, and biometric login via `expo-local-authentication`
- Persist login state across app restarts using `AsyncStorage`

## Lesson Plan

| Duration | What | How or Why |
|---|---|---|
| 10 min | Welcome and recap | Briefly revisit Lesson 2.17: React Navigation, nested navigators, `useFocusEffect`; set context for today's focus on native device features |
| 35 min | Lecture: Native Device Capabilities | Slides: native modules and TurboModules, Expo vs. the manual native way, iOS vs. Android permission models, build-time vs. runtime permissions, overview of Explorer App |
| 5 min | Break | |
| 10 min | Setup | Code-along: create Expo project, install navigation dependencies, create shared style files, run the blank app via Expo Go |
| 12 min | Lab Part 1: Tab navigation shell | Code-along: create stub screens, build `TabNavigator`, wire into `App.js` |
| 12 min | Lab Part 2: Home screen with gallery picker | Code-along: install `expo-image-picker`, configure `app.json`, build `HomeScreen` with `launchImageLibraryAsync`, display selected image |
| 35 min | Lab Part 3: Camera and QR scanning | Code-along: install `expo-camera` and `expo-media-library`, configure permissions, build `CameraScreen` with live preview, flip, `takePhoto`, and `onBarcodeScanned`; create `BarcodeResultScreen` and `AppStackNavigator`; Activity 1: fix repeated scan with `isScanned` and `useFocusEffect` |
| 5 min | Break | |
| 15 min | Lab Part 4: Location and map | Code-along: install `expo-location` and `react-native-maps`, configure `app.json`, build `LocationScreen` with `requestForegroundPermissionsAsync`, `getCurrentPositionAsync`, and `MapView` with `Marker` |
| 25 min | Lab Part 5: Authenticated navigation shell | Code-along: install `expo-local-authentication`, build `AuthContext` with `biometricLogin`, `LoginScreen`, `RegisterScreen`, `SettingsScreen`, `AuthStackNavigator`, and final `App.js` with `NavigationApp` pattern |
| 12 min | Lab Part 6: Persist login | Code-along: install `AsyncStorage`, add `AUTH_KEY`, `restoreSession` `useEffect`, async `login`/`logout`/`biometricLogin`; Activity 2 (optional, if time permits): conditionally show biometric button |
| 15 min | Wrap up and Q&A | Recap objectives, review permission patterns, introduce `expo-secure-store` for production, preview Lesson 2.19 (coaching session) |
| **Total** | | **~180 min** |
