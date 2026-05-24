# Lesson 2.18: Integration of Native Device Capabilities in Cross-Platform Applications

## Overview

- **Duration:** ~2 hours (hands-on lab)
- **Prerequisites:** Lesson 2.17 (React Navigation: tab and stack navigators, `NavigationContainer`, route params, `useFocusEffect`)

## Learning Objectives

By the end of this lesson, you will be able to:

1. **Configure** iOS and Android permissions for native device features using Expo's `app.json` plugin system
2. **Integrate** `expo-camera` and `expo-media-library` to build a camera screen with live preview, photo capture, and QR code scanning
3. **Use** `expo-image-picker` to let users select media from the device library
4. **Implement** location-based features using `expo-location` and display coordinates on a map with `react-native-maps`
5. **Build** an authenticated navigation shell with `AuthContext`, conditional navigator rendering, and biometric login via `expo-local-authentication`
6. **Persist** login state across app restarts using `AsyncStorage`

---

## Introduction

Mobile devices have hardware and OS-level capabilities that web browsers cannot access the same way: the camera, GPS, the photo library, biometric sensors. These features are what make apps like Instagram, Google Maps, and banking apps possible.

In React Native, accessing these features requires two things. First, your app must **declare** which capabilities it intends to use, so the operating system knows to include the relevant permissions at build time. Second, your app must **request** permission from the user at runtime before using the feature.

Expo simplifies both steps. You declare permissions in `app.json`, and Expo injects them into the platform-specific config files (`Info.plist` on iOS, `AndroidManifest.xml` on Android). Then Expo's SDK libraries provide consistent JavaScript APIs for actually using the features, regardless of whether the user is on iOS or Android.

> iOS and Android handle permissions differently. iOS requires a custom message string (a "usage description") for each permission: this is the text shown to the user in the system dialog. Android prompts the user at runtime without needing a custom string. You will see both approaches as you work through this lesson.

In this lesson you will build **Explorer App**: a four-screen tabbed app with a camera, a photo gallery picker, a live location map, and a complete authentication flow including biometric login and persistent sessions.

---

## Setup: Create the Project

Create a fresh Expo app and install all navigation dependencies at once:

```bash
npx create-expo-app --template blank explorer-app
cd explorer-app
npx expo lint
npx expo install @react-navigation/native @react-navigation/bottom-tabs react-native-screens react-native-safe-area-context @expo/vector-icons
npm start
```

Run the app on your emulator or device via Expo Go. Confirm the default screen loads.

Create the following folder structure:

```
explorer-app/
├── App.js
├── app.json
├── context/
├── navigation/
├── screens/
└── styles/
```

Create the shared style files first. All screens will import from them.

`styles/colors.js`:

```js
export const Colors = {
  PRIMARY: "#1971c2",
  PRIMARY_LIGHT_1: "#4dabf7",
  PRIMARY_LIGHT_2: "#e7f5ff",
};
```

`styles/common.js`:

```js
import { StyleSheet } from "react-native";

export const commonStyles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 16,
    backgroundColor: "#fff",
  },
});
```

**Device check:** the blank app loads on the emulator or device.

---

## Part 1: Tab Navigation Shell

Before adding any native features, set up the navigation structure the rest of the lesson builds on. This reuses what you learned in Lesson 2.17.

### Step 1: Create stub screens

Create four minimal screen files. Each just displays its name for now so you can verify the tabs work before filling in the real content.

`screens/HomeScreen.js`:

```jsx
import { Text, View } from "react-native";
import { commonStyles } from "../styles/common";

function HomeScreen() {
  return (
    <View style={commonStyles.container}>
      <Text>Home</Text>
    </View>
  );
}

export default HomeScreen;
```

Create `screens/CameraScreen.js`, `screens/LocationScreen.js`, and `screens/SettingsScreen.js` using the same structure, substituting the label text for each.

### Step 2: Build the tab navigator

Create `navigation/TabNavigator.js`:

```jsx
import { createBottomTabNavigator } from "@react-navigation/bottom-tabs";
import { Ionicons } from "@expo/vector-icons";
import { Colors } from "../styles/colors";
import HomeScreen from "../screens/HomeScreen";
import CameraScreen from "../screens/CameraScreen";
import LocationScreen from "../screens/LocationScreen";
import SettingsScreen from "../screens/SettingsScreen";

const Tab = createBottomTabNavigator();

function TabNavigator() {
  return (
    <Tab.Navigator
      screenOptions={{
        headerStyle: { backgroundColor: Colors.PRIMARY },
        headerTintColor: "#fff",
        tabBarActiveTintColor: Colors.PRIMARY,
      }}
    >
      <Tab.Screen
        name="Home"
        component={HomeScreen}
        options={{
          tabBarIcon: ({ color, size }) => (
            <Ionicons name="home" size={size} color={color} />
          ),
        }}
      />
      <Tab.Screen
        name="Camera"
        component={CameraScreen}
        options={{
          tabBarIcon: ({ color, size }) => (
            <Ionicons name="camera" size={size} color={color} />
          ),
        }}
      />
      <Tab.Screen
        name="Location"
        component={LocationScreen}
        options={{
          tabBarIcon: ({ color, size }) => (
            <Ionicons name="location" size={size} color={color} />
          ),
        }}
      />
      <Tab.Screen
        name="Settings"
        component={SettingsScreen}
        options={{
          tabBarIcon: ({ color, size }) => (
            <Ionicons name="settings" size={size} color={color} />
          ),
        }}
      />
    </Tab.Navigator>
  );
}

export default TabNavigator;
```

Wire it up in `App.js`:

```jsx
import { NavigationContainer } from "@react-navigation/native";
import { StatusBar } from "expo-status-bar";
import TabNavigator from "./navigation/TabNavigator";

export default function App() {
  return (
    <NavigationContainer>
      <TabNavigator />
      <StatusBar style="auto" />
    </NavigationContainer>
  );
}
```

**Device check:** four tabs appear at the bottom with icons and labelled headers. Switching tabs shows each stub screen.

---

## Part 2: Home Screen with Gallery Picker (`expo-image-picker`)

The Home screen is the simplest native integration in the app: opening the device photo library and displaying a selected image. No camera is used here, and no permission needs to be requested before the picker opens: `launchImageLibraryAsync` triggers the system permission dialog automatically the first time it is called.

### Step 1: Install and configure `expo-image-picker`

```bash
npx expo install expo-image-picker
```

Open `app.json` and add the plugin with iOS permission strings. iOS requires these strings to be declared at build time; they are the exact text shown to the user in the system permission dialog.

```json
{
  "expo": {
    "plugins": [
      [
        "expo-image-picker",
        {
          "photosPermission": "The app accesses your photos to let you share them.",
          "cameraPermission": "The app accesses your camera to let you take photos.",
          "microphonePermission": "Allow $(PRODUCT_NAME) to access your microphone"
        }
      ]
    ]
  }
}
```

`$(PRODUCT_NAME)` is a placeholder that Expo replaces with the app name at build time.

### Step 2: Build HomeScreen

Replace the stub content of `screens/HomeScreen.js`:

```jsx
import { useState } from "react";
import { Button, Dimensions, Image, StyleSheet, Text, View } from "react-native";
import * as ImagePicker from "expo-image-picker";
import { commonStyles } from "../styles/common";

const windowWidth = Dimensions.get("window").width;

function HomeScreen() {
  const [image, setImage] = useState(null);

  const pickImage = async () => {
    try {
      const result = await ImagePicker.launchImageLibraryAsync({
        mediaTypes: ["images", "videos"],
        allowsEditing: true,
        aspect: [4, 3],
        quality: 1,
      });
      if (!result.canceled) {
        setImage(result.assets[0].uri);
      }
    } catch (error) {
      alert("Error picking image: " + error.message);
    }
  };

  return (
    <View style={commonStyles.container}>
      <Text>Welcome to the Explorer App</Text>
      <Text style={{ marginBottom: 16 }}>Discover new places and experiences!</Text>
      <Button title="Pick an Image" onPress={pickImage} />
      {image && <Image source={{ uri: image }} style={styles.image} />}
    </View>
  );
}

const styles = StyleSheet.create({
  image: {
    width: windowWidth - 40,
    aspectRatio: 4 / 3,
    borderRadius: 8,
    marginTop: 16,
  },
});

export default HomeScreen;
```

The `result` object from `launchImageLibraryAsync` has this shape:

- `result.canceled`: `true` if the user closed the picker without selecting anything
- `result.assets`: an array of selected items; `result.assets[0].uri` is the local file path of the chosen image
- `allowsEditing: true` and `aspect` activate the built-in crop UI before the image is returned

**Device check:** tapping "Pick an Image" opens the system photo picker. Selecting a photo displays it below the button. On the emulator, use the photos already in the emulator's library.

---

## Part 3: Camera Screen with Live Preview, Photo Capture, and QR Scanning

The Camera screen builds three features in sequence: a live viewfinder with permission handling, a button to capture and save a photo, and QR/barcode scanning that navigates to a result screen.

### Step 1: Install and configure `expo-camera` and `expo-media-library`

```bash
npx expo install expo-camera expo-media-library
```

Add the camera plugin to `app.json` alongside the existing image picker entry:

```json
{
  "expo": {
    "plugins": [
      [
        "expo-image-picker",
        {
          "photosPermission": "The app accesses your photos to let you share them.",
          "cameraPermission": "The app accesses your camera to let you take photos.",
          "microphonePermission": "Allow $(PRODUCT_NAME) to access your microphone"
        }
      ],
      [
        "expo-camera",
        {
          "cameraPermission": "Allow $(PRODUCT_NAME) to access your camera",
          "microphonePermission": "Allow $(PRODUCT_NAME) to access your microphone",
          "recordAudioAndroid": true
        }
      ]
    ]
  }
}
```

### Step 2: Handle permissions and show the live viewfinder

Replace the stub content of `screens/CameraScreen.js`:

```jsx
import { useRef, useState } from "react";
import { Button, StyleSheet, Text, TouchableOpacity, View } from "react-native";
import { CameraView, useCameraPermissions } from "expo-camera";
import { Ionicons } from "@expo/vector-icons";
import { commonStyles } from "../styles/common";
import { Colors } from "../styles/colors";

function CameraScreen() {
  const [facing, setFacing] = useState("back");
  const [permission, requestPermission] = useCameraPermissions();
  const cameraRef = useRef(null);

  if (!permission) {
    return <View />;
  }

  if (!permission.granted) {
    return (
      <View style={styles.permissionContainer}>
        <Text style={styles.permissionMessage}>
          We need your permission to show the camera.
        </Text>
        <Button onPress={requestPermission} title="Grant Permission" />
      </View>
    );
  }

  const toggleCameraFacing = () => {
    setFacing((current) => (current === "back" ? "front" : "back"));
  };

  return (
    <View style={commonStyles.container}>
      <CameraView facing={facing} style={styles.camera} ref={cameraRef} />
      <View style={styles.buttonsContainer}>
        <TouchableOpacity onPress={toggleCameraFacing} style={styles.button}>
          <Text style={styles.buttonText}>Flip Camera</Text>
          <Ionicons name="camera-reverse" size={36} color="white" />
        </TouchableOpacity>
      </View>
    </View>
  );
}

const styles = StyleSheet.create({
  permissionContainer: {
    flex: 1,
    justifyContent: "center",
    alignItems: "center",
  },
  permissionMessage: {
    textAlign: "center",
    marginBottom: 8,
  },
  camera: {
    flex: 1,
    borderRadius: 8,
    overflow: "hidden",
  },
  buttonsContainer: {
    flexDirection: "row",
    justifyContent: "center",
    gap: 20,
    position: "absolute",
    bottom: 64,
    left: 0,
    right: 0,
  },
  button: {
    flexDirection: "row",
    justifyContent: "center",
    alignItems: "center",
    gap: 8,
    backgroundColor: "#1971c280",
    paddingHorizontal: 16,
    paddingVertical: 8,
    borderRadius: 8,
  },
  buttonText: {
    color: "#fff",
    fontSize: 12,
    textTransform: "uppercase",
    fontWeight: "bold",
  },
});

export default CameraScreen;
```

`useCameraPermissions` returns a `[permission, requestPermission]` pair. The two guard clauses at the top handle the two states before the camera can be shown:

- `!permission`: the permission status has not loaded yet (this is brief on first mount). Return an empty view to avoid a flash of incorrect UI.
- `!permission.granted`: the user has not granted access yet, or has denied it. Show a prompt with a button that calls `requestPermission`.

**Device check:** the permission dialog appears on first launch. Accepting shows the live viewfinder. Tapping "Flip Camera" switches between front and back cameras.

On the Android emulator, the camera shows a simulated room scene. On the iOS Simulator, the camera is not available; use a physical device with Expo Go.

### Step 3: Capture a photo and save it to the media library

The camera permission and the media library permission are separate. A user can grant camera access but deny the right to save photos, so always check the media library permission before saving.

Add the `takePhoto` function to `CameraScreen`, just above the `return`:

```jsx
import * as MediaLibrary from "expo-media-library";
import { Alert, Platform } from "react-native";

  const takePhoto = async () => {
    if (!cameraRef.current) return;
    try {
      if (Platform.OS === "ios") {
        const { status } = await MediaLibrary.requestPermissionsAsync();
        if (status !== "granted") {
          Alert.alert(
            "Permission Required",
            "Media Library permission is required to save photos."
          );
          return;
        }
      }
      const photo = await cameraRef.current.takePictureAsync();
      await MediaLibrary.createAssetAsync(photo.uri);
      Alert.alert("Photo Taken", "Photo saved to media library.");
    } catch (error) {
      console.log("Error taking photo:", error);
    }
  };
```

On iOS, media library permission is requested inside `takePhoto` rather than on screen mount. This way the user is not hit with two permission dialogs back to back when they first open the camera screen. On Android, `MediaLibrary` handles permissions at the OS level without needing an explicit request here.

Add a "Take Photo" button alongside the flip button inside `buttonsContainer`:

```jsx
        <TouchableOpacity onPress={takePhoto} style={styles.button}>
          <Text style={styles.buttonText}>Take Photo</Text>
          <Ionicons name="camera" size={36} color="white" />
        </TouchableOpacity>
```

**Device check:** tapping "Take Photo" saves a photo and shows the confirmation alert. Open the device photo library to verify the photo is there.

> **Common mistake:** calling `MediaLibrary.createAssetAsync` before requesting the media library permission on iOS. The call will silently fail or throw. Always check the permission first, even if the camera permission has already been granted; they are independent.

### Step 4: Add QR and barcode scanning

`CameraView` supports barcode scanning via the `onBarcodeScanned` prop. When a handler is passed, the camera scans continuously and calls the handler every time it detects a code, receiving `{ type, data }`.

The scanned result should open a new screen. `BarcodeResultScreen` should not be in the tab bar, so it must live in a stack navigator that wraps the tab navigator. Add it to the `navigation/` folder now.

Create `screens/BarcodeResultScreen.js`:

```jsx
import { Button, StyleSheet, Text, View } from "react-native";
import { commonStyles } from "../styles/common";

function BarcodeResultScreen({ route, navigation }) {
  const { barcodeData } = route.params;

  return (
    <View style={commonStyles.container}>
      <Text style={styles.title}>Barcode Result</Text>
      <Text style={styles.barcodeText}>{barcodeData}</Text>
      <Button title="Go Back" onPress={() => navigation.goBack()} />
    </View>
  );
}

const styles = StyleSheet.create({
  title: { fontSize: 24, fontWeight: "bold", marginBottom: 16 },
  barcodeText: { fontSize: 18, marginBottom: 24 },
});

export default BarcodeResultScreen;
```

Create `navigation/AppStackNavigator.js`. This wraps `TabNavigator` as its first screen (with `headerShown: false` so only one header is visible at a time) and registers `BarcodeResultScreen` as a second screen reachable from anywhere inside the tabs:

```jsx
import { createNativeStackNavigator } from "@react-navigation/native-stack";
import { Colors } from "../styles/colors";
import TabNavigator from "./TabNavigator";
import BarcodeResultScreen from "../screens/BarcodeResultScreen";

const Stack = createNativeStackNavigator();

function AppStackNavigator() {
  return (
    <Stack.Navigator
      screenOptions={{
        headerStyle: { backgroundColor: Colors.PRIMARY },
        headerTintColor: "#fff",
      }}
    >
      <Stack.Screen
        name="Tabs"
        component={TabNavigator}
        options={{ headerShown: false }}
      />
      <Stack.Screen
        name="BarcodeResult"
        component={BarcodeResultScreen}
        options={{ title: "Barcode Result" }}
      />
    </Stack.Navigator>
  );
}

export default AppStackNavigator;
```

Update `App.js` to use `AppStackNavigator` instead of `TabNavigator`:

```jsx
import { NavigationContainer } from "@react-navigation/native";
import { StatusBar } from "expo-status-bar";
import AppStackNavigator from "./navigation/AppStackNavigator";

export default function App() {
  return (
    <NavigationContainer>
      <AppStackNavigator />
      <StatusBar style="auto" />
    </NavigationContainer>
  );
}
```

Now wire up the barcode handler in `CameraScreen`. Because `CameraScreen` is a registered tab screen, the `navigation` prop is automatically provided; add it to the function signature:

```jsx
function CameraScreen({ navigation }) {
```

Add the handler and pass it to `CameraView`:

```jsx
  const handleBarcodeScanned = ({ type, data }) => {
    navigation.navigate("BarcodeResult", { barcodeData: data });
  };
```

Update `CameraView` in the `return`:

```jsx
      <CameraView
        facing={facing}
        style={styles.camera}
        ref={cameraRef}
        onBarcodeScanned={handleBarcodeScanned}
      />
```

**Device check:** point the camera at a QR code. The app navigates to `BarcodeResultScreen` showing the decoded data. Tapping "Go Back" returns to the Camera tab.

> **Instructor note on emulators:** On the Android emulator, use the Extended Controls camera panel to display a QR code in the simulated scene, or mirror a physical device using scrcpy. On the iOS Simulator, use a physical device with Expo Go.

---

## Activity 1: Fix the Repeated Scan Problem

The `onBarcodeScanned` handler fires on every camera frame once a barcode is detected. This means `navigation.navigate` is called many times in rapid succession, which can push multiple copies of `BarcodeResultScreen` onto the stack.

Your task: make `CameraScreen` navigate only once per scan. After the first scan triggers navigation, the scanner should pause until the user returns to the Camera tab.

**Hints:**

1. Add a state variable `isScanned` (boolean, starts `false`). Set it to `true` inside `handleBarcodeScanned` before navigating.
2. When `isScanned` is `true`, pass `undefined` to `onBarcodeScanned` instead of the handler function. Setting the prop to `undefined` pauses scanning.
3. Use `useFocusEffect` (from `@react-navigation/native`) to reset `isScanned` back to `false` each time the screen comes into focus, so the user can scan again after returning.
4. `useFocusEffect` requires its callback to be wrapped in `useCallback`.

<details>
<summary>Reference solution</summary>

```jsx
import { useCallback, useRef, useState } from "react";
import { useFocusEffect } from "@react-navigation/native";

function CameraScreen({ navigation }) {
  const [facing, setFacing] = useState("back");
  const [permission, requestPermission] = useCameraPermissions();
  const [isScanned, setIsScanned] = useState(false);
  const cameraRef = useRef(null);

  useFocusEffect(
    useCallback(() => {
      setIsScanned(false);
    }, [])
  );

  // ... permission guards ...

  const handleBarcodeScanned = ({ type, data }) => {
    setIsScanned(true);
    navigation.navigate("BarcodeResult", { barcodeData: data });
  };

  return (
    <View style={commonStyles.container}>
      <CameraView
        facing={facing}
        style={styles.camera}
        ref={cameraRef}
        onBarcodeScanned={isScanned ? undefined : handleBarcodeScanned}
      />
      {/* ... buttons ... */}
    </View>
  );
}
```

</details>

---

## Part 4: Location Screen with `expo-location` and `react-native-maps`

### Step 1: Install dependencies

```bash
npx expo install expo-location react-native-maps
```

Add the `expo-location` plugin to `app.json`:

```json
      [
        "expo-location",
        {
          "locationAlwaysAndWhenInUsePermission": "Allow the app to use your location."
        }
      ]
```

### Step 2: Build LocationScreen

Unlike the camera, the location permission is requested inside the `getLocation` handler rather than on mount. This way the user only sees the permission dialog when they actively tap the button, not as soon as they open the tab.

Replace the stub content of `screens/LocationScreen.js`:

```jsx
import { useState } from "react";
import {
  ActivityIndicator,
  Button,
  StyleSheet,
  Text,
  View,
} from "react-native";
import * as Location from "expo-location";
import MapView, { Marker } from "react-native-maps";
import { commonStyles } from "../styles/common";
import { Colors } from "../styles/colors";

function LocationScreen() {
  const [location, setLocation] = useState(null);
  const [retrievingLocation, setRetrievingLocation] = useState(false);

  const getLocation = async () => {
    const { status } = await Location.requestForegroundPermissionsAsync();
    if (status !== "granted") {
      alert("Permission to access location was denied.");
      return;
    }
    setRetrievingLocation(true);
    try {
      const retrievedLocation = await Location.getCurrentPositionAsync({});
      setLocation(retrievedLocation);
    } catch (error) {
      alert("Error retrieving location: " + error.message);
    } finally {
      setRetrievingLocation(false);
    }
  };

  return (
    <View style={commonStyles.container}>
      <Button title="Get Location" onPress={getLocation} />
      {location && (
        <Text>
          Your coordinates: {location.coords.latitude},{" "}
          {location.coords.longitude}
        </Text>
      )}
      {retrievingLocation && (
        <ActivityIndicator size="large" color={Colors.PRIMARY} />
      )}
      {location?.coords && (
        <MapView
          style={styles.map}
          initialRegion={{
            latitude: location.coords.latitude,
            longitude: location.coords.longitude,
            latitudeDelta: 0.01,
            longitudeDelta: 0.01,
          }}
        >
          <Marker
            coordinate={{
              latitude: location.coords.latitude,
              longitude: location.coords.longitude,
            }}
            title="You are here"
          />
        </MapView>
      )}
    </View>
  );
}

const styles = StyleSheet.create({
  map: {
    flex: 1,
    width: "100%",
    marginTop: 16,
    borderRadius: 8,
  },
});

export default LocationScreen;
```

Three things to note:

- `requestForegroundPermissionsAsync` requests permission to access location while the app is in the foreground. There is also a `requestBackgroundPermissionsAsync` for apps that need location when minimised, but that is not needed here.
- `finally` is the right place to reset `retrievingLocation` to `false`. It runs whether the `try` block succeeded or `catch` handled an error, so the spinner is always cleared.
- `location?.coords &&` uses optional chaining to guard against rendering `MapView` before location has been set. Without the guard, accessing `location.coords` when `location` is `null` would throw.

**Device check:** tapping "Get Location" shows the permission dialog. After granting, coordinates appear and a map renders centred on the current position with a "You are here" marker.

On the Android emulator, set a simulated location via Extended Controls > Location. On iOS Simulator, use Features > Location > Custom Location.

---

## Part 5: Authenticated Navigation Shell

This section adds a login flow to the app. `AuthContext` holds authentication state using the Context API from Lesson 2.6, and `App.js` renders either the auth screens or the main app depending on whether the user is logged in.

### Step 1: Install `expo-local-authentication`

```bash
npx expo install expo-local-authentication
```

This library provides access to Face ID, fingerprint, and device PIN for biometric authentication.

### Step 2: Create `AuthContext`

Create `context/AuthContext.js`. This is the single source of truth for authentication state. It exposes three functions: `login`, `logout`, and `biometricLogin`.

```jsx
import { createContext, useState } from "react";
import * as LocalAuthentication from "expo-local-authentication";

const AuthContext = createContext();

export function AuthProvider({ children }) {
  const [isAuthenticated, setIsAuthenticated] = useState(false);

  const login = (username, password) => {
    // In a real app: call an API, verify credentials, then store the returned JWT
    setIsAuthenticated(true);
  };

  const logout = () => {
    setIsAuthenticated(false);
  };

  const biometricLogin = async () => {
    const hasHardware = await LocalAuthentication.hasHardwareAsync();
    const isEnrolled = await LocalAuthentication.isEnrolledAsync();
    if (!hasHardware || !isEnrolled) return false;

    const result = await LocalAuthentication.authenticateAsync({
      promptMessage: "Authenticate with Biometrics",
    });
    if (result.success) {
      setIsAuthenticated(true);
      return true;
    }
    return false;
  };

  return (
    <AuthContext.Provider value={{ isAuthenticated, login, logout, biometricLogin }}>
      {children}
    </AuthContext.Provider>
  );
}

export default AuthContext;
```

`biometricLogin` checks two things before attempting authentication:

- `hasHardwareAsync`: does the device have a biometric sensor at all?
- `isEnrolledAsync`: has the user enrolled a fingerprint or face?

Both must be true before calling `authenticateAsync`, which triggers the system prompt. `result.success` is `true` if the user authenticated successfully.

### Step 3: Create the auth screens

Create `screens/LoginScreen.js`:

```jsx
import { useContext, useState } from "react";
import { Button, StyleSheet, Text, TextInput, View } from "react-native";
import { Ionicons } from "@expo/vector-icons";
import AuthContext from "../context/AuthContext";
import { Colors } from "../styles/colors";

function LoginScreen({ navigation }) {
  const [username, setUsername] = useState("");
  const [password, setPassword] = useState("");
  const { login, biometricLogin } = useContext(AuthContext);

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Login</Text>
      <TextInput
        style={styles.input}
        placeholder="Username"
        value={username}
        onChangeText={setUsername}
      />
      <TextInput
        style={styles.input}
        placeholder="Password"
        value={password}
        onChangeText={setPassword}
        secureTextEntry
      />
      <View style={styles.buttonsContainer}>
        <Button title="Login" onPress={() => login(username, password)} />
        <Button
          title="Register"
          onPress={() => navigation.navigate("Register")}
        />
      </View>
      <Ionicons
        name="finger-print"
        size={48}
        color={Colors.PRIMARY}
        onPress={biometricLogin}
      />
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    alignItems: "center",
    justifyContent: "center",
  },
  title: {
    fontSize: 24,
    marginBottom: 24,
  },
  input: {
    width: "80%",
    borderWidth: 1,
    borderColor: "#ccc",
    borderRadius: 4,
    padding: 8,
    marginBottom: 16,
  },
  buttonsContainer: {
    flexDirection: "row",
    gap: 10,
    marginBottom: 24,
  },
});

export default LoginScreen;
```

Create `screens/RegisterScreen.js`. This is a mock; in a real app it would call a registration API, but for now it just shows an alert:

```jsx
import { useState } from "react";
import { Alert, Button, StyleSheet, Text, TextInput, View } from "react-native";

function RegisterScreen({ navigation }) {
  const [username, setUsername] = useState("");
  const [password, setPassword] = useState("");

  const handleRegister = () => {
    Alert.alert(
      "Mock Registration",
      "This is just a mock registration function."
    );
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Register</Text>
      <TextInput
        style={styles.input}
        placeholder="Username"
        value={username}
        onChangeText={setUsername}
      />
      <TextInput
        style={styles.input}
        placeholder="Password"
        value={password}
        onChangeText={setPassword}
        secureTextEntry
      />
      <View style={styles.buttonsContainer}>
        <Button title="Register" onPress={handleRegister} />
        <Button title="Back to Login" onPress={() => navigation.goBack()} />
      </View>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    alignItems: "center",
    justifyContent: "center",
  },
  title: {
    fontSize: 24,
    marginBottom: 24,
  },
  input: {
    width: "80%",
    borderWidth: 1,
    borderColor: "#ccc",
    borderRadius: 4,
    padding: 8,
    marginBottom: 16,
  },
  buttonsContainer: {
    flexDirection: "row",
    gap: 10,
  },
});

export default RegisterScreen;
```

Update `screens/SettingsScreen.js` to show a logout button:

```jsx
import { useContext } from "react";
import { Button, View } from "react-native";
import AuthContext from "../context/AuthContext";
import { commonStyles } from "../styles/common";

function SettingsScreen() {
  const { logout } = useContext(AuthContext);

  return (
    <View style={commonStyles.container}>
      <Button title="Logout" onPress={logout} />
    </View>
  );
}

export default SettingsScreen;
```

### Step 4: Create the auth navigator

Create `navigation/AuthStackNavigator.js`. This wraps the two auth screens in a stack navigator with no header; the Login and Register screens manage their own layout:

```jsx
import { createNativeStackNavigator } from "@react-navigation/native-stack";
import LoginScreen from "../screens/LoginScreen";
import RegisterScreen from "../screens/RegisterScreen";

const Stack = createNativeStackNavigator();

function AuthStackNavigator() {
  return (
    <Stack.Navigator screenOptions={{ headerShown: false }}>
      <Stack.Screen name="Login" component={LoginScreen} />
      <Stack.Screen name="Register" component={RegisterScreen} />
    </Stack.Navigator>
  );
}

export default AuthStackNavigator;
```

### Step 5: Wire up `App.js` with the auth shell

Update `App.js` to wrap everything in `AuthProvider` and conditionally render either the auth flow or the main app based on `isAuthenticated`:

```jsx
import { useContext } from "react";
import { NavigationContainer } from "@react-navigation/native";
import { StatusBar } from "expo-status-bar";
import AuthContext, { AuthProvider } from "./context/AuthContext";
import AppStackNavigator from "./navigation/AppStackNavigator";
import AuthStackNavigator from "./navigation/AuthStackNavigator";

function NavigationApp() {
  const { isAuthenticated } = useContext(AuthContext);

  return (
    <NavigationContainer>
      {isAuthenticated ? <AppStackNavigator /> : <AuthStackNavigator />}
      <StatusBar style="auto" />
    </NavigationContainer>
  );
}

export default function App() {
  return (
    <AuthProvider>
      <NavigationApp />
    </AuthProvider>
  );
}
```

`NavigationApp` is a separate component from `App` for an important reason: `useContext` only works inside the provider it reads from. If `useContext(AuthContext)` were called directly inside `App`, it would be outside `<AuthProvider>` and would always read the default context value, meaning `isAuthenticated` would always be `false`.

By splitting `NavigationApp` out as a child of `<AuthProvider>`, it sits inside the provider tree and reads the real value.

**Device check:** the Login screen appears on launch. Entering any username and password and tapping Login switches to the tab navigator. Tapping Logout on the Settings screen returns to Login. On a device with biometrics enrolled, tapping the fingerprint icon triggers the system biometric prompt and logs in on success.

---

## Part 6: Persisting Login State with `AsyncStorage`

Right now, every time the app restarts, the user must log in again. `AsyncStorage` is a simple key-value store that persists data to disk across app launches. You will use it to save a login flag so returning users go straight to the app.

### Step 1: Install `AsyncStorage`

```bash
npx expo install @react-native-async-storage/async-storage
```

### Step 2: Save and clear the flag in `AuthContext`

Three changes are needed in `context/AuthContext.js`:

1. Save a flag when the user logs in
2. Clear the flag on logout
3. Read the flag on mount and restore the session if it exists

An `isLoading` state prevents the Login screen from flashing briefly on startup while the stored value is being read. Add it to the context value so `App.js` can use it.

Replace the contents of `context/AuthContext.js`:

```jsx
import { createContext, useEffect, useState } from "react";
import AsyncStorage from "@react-native-async-storage/async-storage";
import * as LocalAuthentication from "expo-local-authentication";

const AuthContext = createContext();

const AUTH_KEY = "isLoggedIn";

export function AuthProvider({ children }) {
  const [isAuthenticated, setIsAuthenticated] = useState(false);
  const [isLoading, setIsLoading] = useState(true);

  useEffect(() => {
    const restoreSession = async () => {
      try {
        const value = await AsyncStorage.getItem(AUTH_KEY);
        if (value === "true") {
          setIsAuthenticated(true);
        }
      } catch (error) {
        console.log("Error restoring session:", error);
      } finally {
        setIsLoading(false);
      }
    };
    restoreSession();
  }, []);

  const login = async (username, password) => {
    // In a real app: call an API, verify credentials, store the returned JWT
    await AsyncStorage.setItem(AUTH_KEY, "true");
    setIsAuthenticated(true);
  };

  const logout = async () => {
    await AsyncStorage.removeItem(AUTH_KEY);
    setIsAuthenticated(false);
  };

  const biometricLogin = async () => {
    const hasHardware = await LocalAuthentication.hasHardwareAsync();
    const isEnrolled = await LocalAuthentication.isEnrolledAsync();
    if (!hasHardware || !isEnrolled) return false;

    const result = await LocalAuthentication.authenticateAsync({
      promptMessage: "Authenticate with Biometrics",
    });
    if (result.success) {
      await AsyncStorage.setItem(AUTH_KEY, "true");
      setIsAuthenticated(true);
      return true;
    }
    return false;
  };

  return (
    <AuthContext.Provider
      value={{ isAuthenticated, isLoading, login, logout, biometricLogin }}
    >
      {children}
    </AuthContext.Provider>
  );
}

export default AuthContext;
```

`AUTH_KEY` is a named constant for the storage key string. Using a constant avoids typos from writing `"isLoggedIn"` in multiple places.

`login`, `logout`, and `biometricLogin` are now `async` because `AsyncStorage` operations return promises. The `await` before each `setItem` / `removeItem` ensures the write completes before the state update, so the two stay in sync.

`isLoading` starts as `true` and is set to `false` inside the `finally` block of `restoreSession`. Using `finally` is important: it runs whether `getItem` succeeded or threw an error, so `isLoading` is always eventually cleared.

### Step 3: Show a loading screen while restoring the session

Without a loading state, the Login screen flashes briefly on every launch for already-authenticated users, because React renders before `restoreSession` finishes. Read `isLoading` from `AuthContext` in `NavigationApp` and render an `ActivityIndicator` while the check is in progress:

```jsx
import { ActivityIndicator, View } from "react-native";

function NavigationApp() {
  const { isAuthenticated, isLoading } = useContext(AuthContext);

  if (isLoading) {
    return (
      <View style={{ flex: 1, justifyContent: "center", alignItems: "center" }}>
        <ActivityIndicator size="large" />
      </View>
    );
  }

  return (
    <NavigationContainer>
      {isAuthenticated ? <AppStackNavigator /> : <AuthStackNavigator />}
      <StatusBar style="auto" />
    </NavigationContainer>
  );
}
```

**Device check:** log in, then close and reopen the app. The app should open directly to the tab navigator without the Login screen appearing. Tap Logout on the Settings screen, close the app, and reopen it; the Login screen should appear again.

> **Production note:** This lesson stores a simple boolean flag. In a real app, your login API returns a JWT (JSON Web Token). Store the token with `AsyncStorage.setItem(AUTH_KEY, token)` and send it in the `Authorization` header of subsequent API requests. For higher security, especially for sensitive data like tokens that should survive device backups being compromised, use `expo-secure-store` instead of `AsyncStorage`. It stores data in the device's secure enclave (Keychain on iOS, Keystore on Android).

---

## Activity 2: Conditionally Show the Biometric Login Button

The fingerprint icon in `LoginScreen` is always visible, even on emulators or devices where biometrics are not available. On a device with no biometric hardware, tapping it does nothing, which is confusing for users.

Your task: check whether biometric login is actually available and only show the fingerprint icon when it is.

**Hints:**

1. Import `* as LocalAuthentication from "expo-local-authentication"` in `LoginScreen.js`
2. Add a state variable `biometricAvailable` (boolean, starts `false`)
3. In a `useEffect`, call `LocalAuthentication.hasHardwareAsync()` and `LocalAuthentication.isEnrolledAsync()`. Set `biometricAvailable` to `true` only if both return `true`
4. Conditionally render the `Ionicons` fingerprint icon based on `biometricAvailable`

<details>
<summary>Reference solution</summary>

```jsx
import { useContext, useEffect, useState } from "react";
import * as LocalAuthentication from "expo-local-authentication";

function LoginScreen({ navigation }) {
  const [username, setUsername] = useState("");
  const [password, setPassword] = useState("");
  const [biometricAvailable, setBiometricAvailable] = useState(false);
  const { login, biometricLogin } = useContext(AuthContext);

  useEffect(() => {
    const checkBiometrics = async () => {
      const hasHardware = await LocalAuthentication.hasHardwareAsync();
      const isEnrolled = await LocalAuthentication.isEnrolledAsync();
      setBiometricAvailable(hasHardware && isEnrolled);
    };
    checkBiometrics();
  }, []);

  return (
    <View style={styles.container}>
      {/* ... inputs and buttons ... */}
      {biometricAvailable && (
        <Ionicons
          name="finger-print"
          size={48}
          color={Colors.PRIMARY}
          onPress={biometricLogin}
        />
      )}
    </View>
  );
}
```

</details>

---

## Bonus Challenges

Work on as many as you can. They are listed in order of difficulty. No solutions are provided.

### Challenge 1: Login validation

The `login` function currently accepts any input, including empty strings. Add validation: if either `username` or `password` is empty, show an `Alert` and return without writing to `AsyncStorage` or setting `isAuthenticated`.

### Challenge 2: Camera flash toggle

In `CameraScreen`, add a button that cycles through flash modes: `"off"` → `"on"` → `"auto"` → back to `"off"`. Pass the current mode to the `flash` prop on `CameraView`. Display the current mode using an appropriate `Ionicons` icon (`"flash-off"`, `"flash"`, `"flash-outline"`).

### Challenge 3: Image preview before saving

After `takePictureAsync`, store the photo URI in state and display a preview overlay on top of `CameraView`. Add "Save" and "Discard" buttons on the overlay. Only call `MediaLibrary.createAssetAsync` when the user taps "Save"; "Discard" clears the preview and returns to the live viewfinder.

### Challenge 4: Live location tracking

Instead of fetching location once with `getCurrentPositionAsync`, use `Location.watchPositionAsync` to subscribe to continuous updates. Update the map region and marker each time a new position is received. When the component unmounts, call `.remove()` on the subscription object returned by `watchPositionAsync` to stop tracking.

---

## Summary

- Expo's `app.json` plugin system is the single place to declare native permissions for both iOS and Android. iOS additionally requires a usage description string for each permission.
- `useCameraPermissions` is a hook that returns the current permission status and a function to request it. Use it for features that are needed immediately when the screen mounts. For features triggered by user action, like getting location, request permission imperatively inside the handler instead.
- Nesting a `TabNavigator` inside a `StackNavigator` is the standard pattern for screens that should be reachable from any tab but should not appear in the tab bar itself.
- Conditional navigator rendering based on `AuthContext` is the idiomatic React Navigation pattern for auth flows: render the auth stack or the app stack, not individual hidden screens.
- `expo-local-authentication` wraps Face ID, fingerprint, and device PIN behind a single `authenticateAsync` call. Always check `hasHardwareAsync` and `isEnrolledAsync` before attempting authentication.
- `AsyncStorage` persists key-value data to disk across app launches. Use `isLoading` state to prevent a flash of the Login screen while a stored session is being restored on startup. For sensitive data in production, use `expo-secure-store` instead.

---

## Additional Resources

- [Expo Camera docs](https://docs.expo.dev/versions/latest/sdk/camera/)
- [Expo Image Picker docs](https://docs.expo.dev/versions/latest/sdk/imagepicker/)
- [Expo Media Library docs](https://docs.expo.dev/versions/latest/sdk/media-library/)
- [Expo Location docs](https://docs.expo.dev/versions/latest/sdk/location/)
- [Expo Local Authentication docs](https://docs.expo.dev/versions/latest/sdk/local-authentication/)
- [AsyncStorage docs](https://react-native-async-storage.github.io/async-storage/)
- [Expo SecureStore docs](https://docs.expo.dev/versions/latest/sdk/securestore/)
- [react-native-maps docs](https://github.com/react-native-maps/react-native-maps)
- [React Navigation: Authentication Flows](https://reactnavigation.org/docs/auth-flow)
