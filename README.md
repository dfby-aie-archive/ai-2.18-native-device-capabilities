# 2.18 Integration of Native Device Capabilities in Cross-Platform Applications

## Lesson Overview

This lesson teaches learners how to access native device hardware and OS features in a React Native app built with Expo. Starting from a blank Expo project, learners build Explorer App: a four-tab app with a gallery image picker, a live camera view with photo capture and QR code scanning, a GPS location screen with a map, and a complete authentication flow including biometric login and persistent sessions backed by AsyncStorage.

## Dependencies

- [Self Studies](./studies.md)
- [Lesson](./lesson.md)
- [Assessment](./assessment.md)
- [Assignment](./assignment.md)

## Lesson Objectives

- Configure iOS and Android permissions for native device features using Expo's `app.json` plugin system
- Integrate `expo-camera`, `expo-media-library`, and `expo-image-picker` to capture photos, save them to the device library, and scan QR codes
- Implement location-based features using `expo-location` and display coordinates on an interactive map with `react-native-maps`
- Build an authenticated navigation shell with `AuthContext`, conditional navigator rendering, and biometric login via `expo-local-authentication`
- Persist login state across app restarts using `AsyncStorage`, with an `isLoading` guard to prevent the Login screen from flashing on startup

## Lesson Plan

| Duration | What | How or Why |
|---|---|---|
| 10 min | Welcome and recap | Briefly revisit Lesson 2.17: React Navigation, nested navigators, `useFocusEffect`; introduce Explorer App and what learners will build |
| 15 min | Setup and tab shell | Code-along: create Expo project, install nav dependencies, create stub screens and shared styles, build `TabNavigator` and wire into `App.js` |
| 20 min | Part 2: Image Picker | Code-along: install `expo-image-picker`, configure `app.json`, build `HomeScreen` with `launchImageLibraryAsync`, display selected image |
| 35 min | Part 3: Camera and QR scanning | Code-along: install `expo-camera` and `expo-media-library`, configure permissions, build `CameraScreen` with live preview, flip, `takePhoto`, and `onBarcodeScanned`; create `BarcodeResultScreen` and `AppStackNavigator`; Activity 1: fix repeated scan problem with `isScanned` and `useFocusEffect` |
| 20 min | Part 4: Location and map | Code-along: install `expo-location` and `react-native-maps`, configure `app.json`, build `LocationScreen` with `requestForegroundPermissionsAsync`, `getCurrentPositionAsync`, and `MapView` with `Marker` |
| 15 min | Part 5: Auth shell | Code-along: install `expo-local-authentication`, build `AuthContext` with `biometricLogin`, `LoginScreen`, `RegisterScreen`, `SettingsScreen`, `AuthStackNavigator`, and final `App.js` with `NavigationApp` pattern |
| 20 min | Part 6: Persist login | Code-along: install `AsyncStorage`, add `AUTH_KEY`, `isLoading` state, `restoreSession` `useEffect`, async `login`/`logout`/`biometricLogin`; update `NavigationApp` with `ActivityIndicator` guard; Activity 2: conditionally show biometric button |
| 15 min | Wrap up and Q&A | Recap objectives, review permission patterns, introduce `expo-secure-store` for production, preview Lesson 2.19 (coaching session) |
| **Total** | | **~150 min** |
