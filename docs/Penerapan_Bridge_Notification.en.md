# 🔔 Push Notifications Integration Tutorial - React Native/Expo

## 📋 Table of Contents
1. [Architecture Explanation](#architecture-explanation)
2. [Expo Push Notifications Setup](#expo-push-notifications-setup)
3. [Laravel Backend Integration](#laravel-backend-integration)
4. [React Native Code](#react-native-code)
5. [Testing](#testing)

---

## 🏗️ Architecture Explanation

### ❌ The WebView Problem
**WebView CANNOT receive native push notifications!**

WebView only displays a website inside the application, so:
- ❌ It has no access to native notification APIs
- ❌ It cannot register for push notifications
- ❌ It cannot display notification badges
- ❌ It cannot receive notifications when the app is closed

### ✅ Solution: Native Push Notifications

```
+-----------------+
|   Laravel API   | --+
+-----------------+   |
                      | Send notification
                      v
+-----------------------------+
|  Expo Push Notification     |
|        Service              |
+-----------------------------+
                      |
                      | Deliver to device
                      v
+-----------------------------+
|  React Native App           |
|  (NOT WebView)              |
|  - Receives notification    |
|  - Shows native alert       |
|  - Updates badge count      |
+-----------------------------+
```

---

## 📱 Expo Push Notifications Setup

### Step 1: Install Dependencies

```bash
cd /path/to/your/expo-app

# Install required packages
npx expo install expo-notifications expo-device expo-constants
```

### Step 2: Configure app.json

```json
{
  "expo": {
    "name": "Presensi GSG",
    "slug": "presensi-gsg",
    "version": "1.0.0",
    "orientation": "portrait",
    "icon": "./assets/icon.png",
    "userInterfaceStyle": "light",
    "splash": {
      "image": "./assets/splash.png",
      "resizeMode": "contain",
      "backgroundColor": "#ffffff"
    },
    "ios": {
      "supportsTablet": true,
      "bundleIdentifier": "com.grahasarana.presensi"
    },
    "android": {
      "adaptiveIcon": {
        "foregroundImage": "./assets/adaptive-icon.png",
        "backgroundColor": "#ffffff"
      },
      "package": "com.grahasarana.presensi",
      "googleServicesFile": "./google-services.json",  // <- Add this for Firebase
      "permissions": [
        "NOTIFICATIONS",
        "VIBRATE"
      ]
    },
    "plugins": [
      [
        "expo-notifications",
        {
          "icon": "./assets/notification-icon.png",
          "color": "#696cff",
          "sounds": ["./assets/notification-sound.wav"]
        }
      ]
    ]
  }
}
```

---

## 📝 React Native Code

### Step 1: Create Notification Service

Create file: `services/NotificationService.js`

```javascript
import * as Notifications from 'expo-notifications';
import * as Device from 'expo-device';
import Constants from 'expo-constants';
import { Platform } from 'react-native';
import axios from 'axios';

const API_URL = 'https://absensi.grahasaranagresik.com/api';

// Configure how notifications are handled when app is in foreground
Notifications.setNotificationHandler({
  handleNotification: async () => ({
    shouldShowAlert: true,
    shouldPlaySound: true,
    shouldSetBadge: true,
  }),
});

class NotificationService {
  constructor() {
    this.expoPushToken = null;
    this.notificationListener = null;
    this.responseListener = null;
  }

  /**
   * Register for push notifications
   * Call this on app startup after user login
   */
  async registerForPushNotifications(employeeId, authToken) {
    if (!Device.isDevice) {
      alert('Must use physical device for Push Notifications');
      return null;
    }

    // Check existing permissions
    const { status: existingStatus } = await Notifications.getPermissionsAsync();
    let finalStatus = existingStatus;

    // Request permissions if not granted
    if (existingStatus !== 'granted') {
      const { status } = await Notifications.requestPermissionsAsync();
      finalStatus = status;
    }

    if (finalStatus !== 'granted') {
      alert('Failed to get push token for push notification!');
      return null;
    }

    // Get Expo Push Token
    const projectId = Constants.expoConfig?.extra?.eas?.projectId ?? Constants.easConfig?.projectId;
    
    const token = await Notifications.getExpoPushTokenAsync({
      projectId, // Required for EAS builds
    });

    this.expoPushToken = token.data;

    // Save token to backend
    await this.saveTokenToBackend(employeeId, token.data, authToken);

    // Configure Android notification channel
    if (Platform.OS === 'android') {
      await Notifications.setNotificationChannelAsync('default', {
        name: 'default',
        importance: Notifications.AndroidImportance.MAX,
        vibrationPattern: [0, 250, 250, 250],
        lightColor: '#696cff',
      });
    }

    return token.data;
  }

  /**
   * Save device token to Laravel backend
   */
  async saveTokenToBackend(employeeId, expoPushToken, authToken) {
    try {
      const response = await axios.post(
        `${API_URL}/device-tokens`,
        {
          employee_id: employeeId,
          token: expoPushToken,
          platform: Platform.OS,
          device_name: `${Device.brand} ${Device.modelName}`,
        },
        {
          headers: {
            'Authorization': `Bearer ${authToken}`,
            'Content-Type': 'application/json',
          },
        }
      );

      console.log('✅ Token saved to backend:', response.data);
      return response.data;
    } catch (error) {
      console.error('❌ Failed to save token:', error.response?.data || error.message);
      throw error;
    }
  }

  /**
   * Setup notification listeners
   * Call this in your root App component
   */
  setupNotificationListeners(onNotificationReceived, onNotificationTapped) {
    // Listener for notifications received while app is in foreground
    this.notificationListener = Notifications.addNotificationReceivedListener(
      (notification) => {
        console.log('📩 Notification received:', notification);
        if (onNotificationReceived) {
          onNotificationReceived(notification);
        }
      }
    );

    // Listener for when user taps on notification
    this.responseListener = Notifications.addNotificationResponseReceivedListener(
      (response) => {
        console.log('👆 Notification tapped:', response);
        if (onNotificationTapped) {
          onNotificationTapped(response);
        }
      }
    );
  }

  /**
   * Clean up listeners
   */
  removeNotificationListeners() {
    if (this.notificationListener) {
      Notifications.removeNotificationSubscription(this.notificationListener);
    }
    if (this.responseListener) {
      Notifications.removeNotificationSubscription(this.responseListener);
    }
  }

  /**
   * Get current badge count
   */
  async getBadgeCount() {
    return await Notifications.getBadgeCountAsync();
  }

  /**
   * Set badge count
   */
  async setBadgeCount(count) {
    await Notifications.setBadgeCountAsync(count);
  }

  /**
   * Clear all notifications
   */
  async clearAllNotifications() {
    await Notifications.dismissAllNotificationsAsync();
    await this.setBadgeCount(0);
  }

  /**
   * Fetch notifications from backend
   */
  async fetchNotifications(employeeId, authToken) {
    try {
      const response = await axios.get(
        `${API_URL}/notifications?employee_id=${employeeId}`,
        {
          headers: {
            'Authorization': `Bearer ${authToken}`,
          },
        }
      );
      return response.data;
    } catch (error) {
      console.error('Failed to fetch notifications:', error);
      return [];
    }
  }

  /**
   * Mark notification as read
   */
  async markAsRead(notificationId, authToken) {
    try {
      await axios.post(
        `${API_URL}/notifications/${notificationId}/read`,
        {},
        {
          headers: {
            'Authorization': `Bearer ${authToken}`,
          },
        }
      );
    } catch (error) {
      console.error('Failed to mark notification as read:', error);
    }
  }
}

export default new NotificationService();
```

---

### Step 2: Integrate in App.js

```javascript
import React, { useState, useEffect, useRef } from 'react';
import { Text, View, Button, Platform, Alert } from 'react-native';
import * as Notifications from 'expo-notifications';
import NotificationService from './services/NotificationService';

export default function App() {
  const [expoPushToken, setExpoPushToken] = useState('');
  const [notification, setNotification] = useState(false);
  const [employeeId, setEmployeeId] = useState(null);
  const [authToken, setAuthToken] = useState(null);

  useEffect(() => {
    // TODO: Get employeeId and authToken from your login process
    // For now, hardcode for testing
    const testEmployeeId = 1;
    const testAuthToken = 'your-auth-token-here';
    
    setEmployeeId(testEmployeeId);
    setAuthToken(testAuthToken);

    // Register for push notifications after login
    if (testEmployeeId && testAuthToken) {
      registerNotifications(testEmployeeId, testAuthToken);
    }

    // Setup listeners
    NotificationService.setupNotificationListeners(
      handleNotificationReceived,
      handleNotificationTapped
    );

    // Cleanup on unmount
    return () => {
      NotificationService.removeNotificationListeners();
    };
  }, []);

  const registerNotifications = async (empId, token) => {
    try {
      const expoPushToken = await NotificationService.registerForPushNotifications(
        empId,
        token
      );
      setExpoPushToken(expoPushToken);
      console.log('Expo Push Token:', expoPushToken);
    } catch (error) {
      console.error('Failed to register notifications:', error);
    }
  };

  const handleNotificationReceived = (notification) => {
    setNotification(notification);
    // Update your UI, show in-app notification, etc.
    Alert.alert(
      notification.request.content.title || 'New Notification',
      notification.request.content.body || ''
    );
  };

  const handleNotificationTapped = (response) => {
    const data = response.notification.request.content.data;
    
    // Navigate to specific screen based on notification data
    if (data.screen) {
      // navigation.navigate(data.screen, data.params);
    }
  };

  const sendTestNotification = async () => {
    await Notifications.scheduleNotificationAsync({
      content: {
        title: "Test Notification",
        body: 'This is a test notification from your app!',
        data: { screen: 'Home' },
      },
      trigger: { seconds: 2 },
    });
  };

  return (
    <View style={{ flex: 1, alignItems: 'center', justifyContent: 'center' }}>
      <Text>Your App!</Text>
      <Text>Push Token: {expoPushToken}</Text>
      
      <Button
        title="Send Test Notification"
        onPress={sendTestNotification}
      />
      
      {notification && (
        <View>
          <Text>Last Notification:</Text>
          <Text>{JSON.stringify(notification, null, 2)}</Text>
        </View>
      )}
    </View>
  );
}
```

---

## 🔧 Laravel Backend Integration

### Step 1: Create API Route for Device Tokens

Add to `routes/api.php`:

```php
// Device Token Management
Route::middleware(['auth:sanctum'])->group(function () {
    Route::post('/device-tokens', [DeviceTokenController::class, 'store']);
    Route::delete('/device-tokens/{token}', [DeviceTokenController::class, 'destroy']);
    
    // Notifications
    Route::get('/notifications', [NotificationController::class, 'index']);
    Route::post('/notifications/{id}/read', [NotificationController::class, 'markAsRead']);
});
```

### Step 2: Create API Controllers

Create `app/Http/Controllers/Api/DeviceTokenController.php`:

```php
<?php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Models\DeviceToken;
use Illuminate\Http\Request;

class DeviceTokenController extends Controller
{
    public function store(Request $request)
    {
        $validated = $request->validate([
            'employee_id' => 'required|exists:employees,id',
            'token' => 'required|string',
            'platform' => 'required|in:android,ios,web',
            'device_name' => 'nullable|string',
        ]);

        // Delete old token if exists
        DeviceToken::where('employee_id', $validated['employee_id'])
            ->where('platform', $validated['platform'])
            ->delete();

        // Create new token
        $deviceToken = DeviceToken::create([
            'employee_id' => $validated['employee_id'],
            'token' => $validated['token'],
            'platform' => $validated['platform'],
            'device_name' => $validated['device_name'],
            'last_used_at' => now(),
        ]);

        return response()->json([
            'success' => true,
            'message' => 'Device token registered successfully',
            'data' => $deviceToken,
        ]);
    }

    public function destroy($token)
    {
        DeviceToken::where('token', $token)->delete();

        return response()->json([
            'success' => true,
            'message' => 'Device token removed successfully',
        ]);
    }
}
```

Create `app/Http/Controllers/Api/NotificationController.php`:

```php
<?php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Models\Notification;
use Illuminate\Http\Request;

class NotificationController extends Controller
{
    public function index(Request $request)
    {
        $employeeId = $request->input('employee_id') ?? $request->user()->id;
        
        $notifications = Notification::where('employee_id', $employeeId)
            ->orderBy('created_at', 'desc')
            ->limit(50)
            ->get();

        $unreadCount = Notification::where('employee_id', $employeeId)
            ->where('read', false)
            ->count();

        return response()->json([
            'success' => true,
            'data' => $notifications,
            'unread_count' => $unreadCount,
        ]);
    }

    public function markAsRead($id)
    {
        $notification = Notification::findOrFail($id);
        $notification->update(['read' => true]);

        return response()->json([
            'success' => true,
            'message' => 'Notification marked as read',
        ]);
    }
}
```

### Step 3: Update PushNotificationService for Expo

Update `app/Services/PushNotificationService.php`, add a method to send via Expo:

```php
/**
 * Send push notification via Expo Push Notification Service
 */
private function sendExpoNotification(string $expoPushToken, string $title, string $message, array $data = []): bool
{
    try {
        $response = Http::post('https://exp.host/--/api/v2/push/send', [
            'to' => $expoPushToken,
            'title' => $title,
            'body' => $message,
            'data' => $data,
            'sound' => 'default',
            'badge' => 1,
            'priority' => 'high',
        ]);

        return $response->successful();
    } catch (\Exception $e) {
        Log::error('Failed to send Expo notification', [
            'error' => $e->getMessage(),
            'token' => $expoPushToken,
        ]);
        return false;
    }
}
```

Update the `sendToUser` method to include Expo push:

```php
public function sendToUser(
    int $userId,
    string $title,
    string $message,
    string $type = 'alert',
    string $priority = 'default',
    array $data = []
): ?Notification {
    try {
        // Create notification in database
        $notification = Notification::create([
            'employee_id' => $userId,
            'type' => $type,
            'message' => $message,
            'data' => array_merge($data, [
                'title' => $title,
                'priority' => $priority,
                'timestamp' => now()->toIso8601String(),
            ]),
            'read' => false,
        ]);

        // Send push notifications to all user's devices
        $deviceTokens = DeviceToken::where('employee_id', $userId)
            ->where('last_used_at', '>=', now()->subDays(30))
            ->get();

        foreach ($deviceTokens as $deviceToken) {
            $this->sendExpoNotification(
                $deviceToken->token,
                $title,
                $message,
                array_merge($data, [
                    'notification_id' => $notification->id,
                    'type' => $type,
                ])
            );
        }

        return $notification;
    } catch (\Exception $e) {
        Log::error('Failed to send notification to user', [
            'employee_id' => $userId,
            'error' => $e->getMessage(),
        ]);
        return null;
    }
}
```

---

## 🧪 Testing

### 1. Test with Expo Go (Development)

```bash
cd /path/to/your/expo-app
npx expo start
```

Scan the QR code with the Expo Go app on your smartphone.

### 2. Test Notification from Laravel

Open the admin panel in the browser and send a notification to a specific employee.

### 3. Build Production App

```bash
# Build for Android
eas build --platform android

# Build for iOS
eas build --platform ios
```

---

## 📌 Important Notes

### ⚠️ Known Issues

1. **Expo Go Limitations:**
   - Notifications work but may have delays
   - Better to use development build or production build

2. **iOS Specific:**
   - Requires Apple Developer Account ($99/year)
   - Need to configure APNs (Apple Push Notification service)

3. **Android Specific:**
   - Need Firebase Cloud Messaging (FCM) configuration
   - Download `google-services.json` from Firebase Console

### 🔐 Security

- Never expose Expo Push Token to public
- Always validate API requests with authentication
- Use HTTPS for all API calls

---

## 🎯 Summary

### What You Need to Do:

1. ✅ **Remove WebView** - Use native React Native components
2. ✅ **Install Expo Notifications** - `npx expo install expo-notifications`
3. ✅ **Copy NotificationService.js** - From this guide
4. ✅ **Update App.js** - Integrate notification setup
5. ✅ **Create API Endpoints** - For device tokens in Laravel
6. ✅ **Update PushNotificationService** - Support Expo push
7. ✅ **Test on Real Device** - Build and install app

### Next Steps:

1. Start with a development build
2. Test notifications thoroughly
3. Build production APK/IPA
4. Deploy to Play Store / App Store

---

**Good luck! 🚀**

If you have any questions or problems during implementation, feel free to ask!
