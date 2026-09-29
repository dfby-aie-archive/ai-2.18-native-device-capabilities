# Optional Assignment: Field Notes App

## Overview

- **Lesson:** Integration of Native Device Capabilities in Cross-Platform Applications / 2.18
- **Type:** Optional Take-Home Assignment
- **Estimated Time:** 2–3 hours
- **Due:** Before next lesson
- **Submission:** GitHub repository link or ZIP file

## Learning Objectives Covered

This assignment reinforces:

- Requesting and handling native permissions for camera, media library, and location
- Using `expo-camera`, `expo-image-picker`, and `expo-location` to access device hardware
- Persisting data across app restarts using `AsyncStorage`
- Building an authenticated navigation shell with `AuthContext` and conditional navigator rendering

## Assignment Description

Build a **Field Notes** app that lets users record observations in the field. Each note consists of a photo (taken with the camera or selected from the gallery), the device's GPS coordinates at the time of capture, and a short text description. Notes are persisted locally so they survive app restarts. The app is protected by a simple login screen.

### What You Will Build

- A **Login screen** that accepts any non-empty username and password and saves the session to `AsyncStorage`
- A **Notes List screen** (Home tab) showing all saved notes with a thumbnail, description, and coordinates
- A **Add Note screen** (Camera tab) that lets the user take a photo or pick from the gallery, captures the current location, and accepts a short description before saving
- A **Settings screen** with a Logout button that clears the session from `AsyncStorage`

## Requirements

### Core Requirements

#### 1. Project Setup

- [ ] Create a new Expo project: `npx create-expo-app@latest --template blank@sdk-57 FieldNotesApp`
- [ ] Install navigation dependencies:
  ```bash
  npx expo install @react-navigation/native @react-navigation/bottom-tabs @react-navigation/native-stack react-native-screens react-native-safe-area-context @react-native-vector-icons/ionicons
  ```
- [ ] Install native feature dependencies:
  ```bash
  npx expo install expo-camera expo-media-library expo-image-picker expo-location @react-native-async-storage/async-storage
  ```
- [ ] Add all required permission entries to `app.json`
- [ ] Confirm the app runs on your device or emulator

#### 2. Authentication

- [ ] `AuthContext` manages `isAuthenticated` state
- [ ] On startup, `AsyncStorage.getItem` restores the session
- [ ] `login(username, password)` validates that neither field is empty, then writes to `AsyncStorage` and sets `isAuthenticated` to `true`
- [ ] `logout` removes the key from `AsyncStorage` and sets `isAuthenticated` to `false`
- [ ] The navigator tree renders either the auth stack or the app tab navigator based on `isAuthenticated`

#### 3. Navigation Structure

```
NavigationContainer
  AuthStackNavigator (when not authenticated)
    Stack.Screen: "Login" → LoginScreen
  AppTabNavigator (when authenticated)
    Tab.Screen: "Notes" → NotesListScreen
    Tab.Screen: "Add Note" → AddNoteScreen
    Tab.Screen: "Settings" → SettingsScreen
```

- [ ] All tabs must have icons from `@react-native-vector-icons/ionicons`
- [ ] Apply a consistent brand colour to the tab bar and header using `screenOptions`

#### 4. Add Note Screen

- [ ] Provide two options for selecting a photo: "Take Photo" (using `expo-camera`) and "Pick from Gallery" (using `expo-image-picker`)
- [ ] After a photo is selected or taken, automatically fetch the current GPS coordinates using `expo-location`
- [ ] Display a preview of the selected photo, the fetched coordinates, and a `TextInput` for a short description
- [ ] A "Save Note" button stores the note object (photo URI, latitude, longitude, description, timestamp) using `AsyncStorage`
- [ ] Handle permission denials gracefully with an informative message

#### 5. Notes List Screen

- [ ] On mount, read all saved notes from `AsyncStorage` and display them in a list
- [ ] Each list item shows the photo thumbnail (using `Image`), the description, and the formatted coordinates
- [ ] If there are no notes, show a placeholder message such as "No notes yet. Tap Add Note to get started."
- [ ] The list refreshes each time the Notes tab gains focus (use `useFocusEffect`)

#### 6. Settings Screen

- [ ] Display the current username
- [ ] Provide a Logout button that calls `logout` from `AuthContext`

### Data Format

Store notes as a JSON array under a single AsyncStorage key, for example `"fieldNotes"`. Each note object should have this shape:

```js
{
  id: Date.now().toString(),
  photoUri: "file://...",
  latitude: 1.3521,
  longitude: 103.8198,
  description: "Old oak tree near the river",
  timestamp: "2026-05-25T10:30:00.000Z",
}
```

Use `JSON.stringify` when writing and `JSON.parse` when reading. Append new notes to the existing array rather than overwriting it.

## Bonus Challenges

### Easy

- [ ] Show a loading spinner while the GPS coordinates are being fetched after a photo is captured
- [ ] Format the timestamp on each note card as a human-readable date string using `new Date(timestamp).toLocaleString()`
- [ ] Add an `isLoading` state to `AuthContext` so the Login screen does not flash briefly on startup while the session is being restored from `AsyncStorage`

### Medium

- [ ] Add a delete button to each note on the Notes List screen. Removing a note should update `AsyncStorage` immediately.
- [ ] Add a map to the Add Note screen using `react-native-maps` that shows a marker at the fetched coordinates before the note is saved

### Hard

- [ ] Add biometric login using `expo-local-authentication`. Show the fingerprint icon on the Login screen only when both `hasHardwareAsync` and `isEnrolledAsync` return `true`.
- [ ] Add a Map View tab that shows all saved notes as markers on a single map. Tapping a marker shows the note's description in a callout.

## Submission

- [ ] Push your project to a GitHub repository
- [ ] Include a brief `README.md` with: how to install and run the app, a screenshot or screen recording of the main features working, and any bonus challenges you completed

## Resources

- [Expo Camera docs](https://docs.expo.dev/versions/latest/sdk/camera/)
- [Expo Image Picker docs](https://docs.expo.dev/versions/latest/sdk/imagepicker/)
- [Expo Location docs](https://docs.expo.dev/versions/latest/sdk/location/)
- [AsyncStorage docs](https://react-native-async-storage.github.io/async-storage/)
- [react-native-maps docs](https://github.com/react-native-maps/react-native-maps)
- [Expo Local Authentication docs](https://docs.expo.dev/versions/latest/sdk/local-authentication/)
- [Expo Vector Icons](https://icons.expo.fyi)
