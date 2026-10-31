# Tutorial: Integrasi Notifikasi di Aplikasi React Native/Expo

> **Tutorial lengkap** untuk mengupdate aplikasi React Native/Expo Anda agar dapat menerima dan menampilkan push notifications dari Laravel web app.

## Prerequisites

Pastikan Anda sudah install packages berikut di project React Native/Expo:

```bash
npx expo install expo-notifications
npx expo install expo-device
npx expo install react-native-webview
```

## Arsitektur Notifikasi

```
Laravel Web App
       ↓ (postMessage)
React Native WebView
       ↓ (onMessage handler)
Expo Notifications
       ↓
Push Notification Display
```

---

## Langkah 1: Setup Permissions

### File: `app.json`

Tambahkan permissions untuk notifications:

```json
{
  "expo": {
    "plugins": [
      [
        "expo-notifications",
        {
          "icon": "./assets/notification-icon.png",
          "color": "#13c8ec",
          "sounds": ["./assets/notification-sound.wav"]
        }
      ]
    ],
    "notification": {
      "icon": "./assets/notification-icon.png",
      "color": "#13c8ec"
    },
    "android": {
      "adaptiveIcon": {
        "foregroundImage": "./assets/adaptive-icon.png",
        "backgroundColor": "#ffffff"
      },
      "permissions": [
        "NOTIFICATIONS",
        "RECEIVE_BOOT_COMPLETED",
        "VIBRATE"
      ]
    },
    "ios": {
      "infoPlist": {
        "UIBackgroundModes": ["remote-notification"]
      }
    }
  }
}
```

---

## Langkah 2: Buat Notification Handler

### File: `services/NotificationService.js`

```javascript
import * as Notifications from 'expo-notifications';
import * as Device from 'expo-device';
import { Platform } from 'react-native';

// Configure notification behavior
Notifications.setNotificationHandler({
  handleNotification: async () => ({
    shouldShowAlert: true,
    shouldPlaySound: true,
    shouldSetBadge: true,
  }),
});

class NotificationService {
  /**
   * Request notification permissions
   */
  async requestPermissions() {
    if (!Device.isDevice) {
      console.warn('Must use physical device for push notifications');
      return false;
    }

    const { status: existingStatus } = await Notifications.getPermissionsAsync();
    let finalStatus = existingStatus;
    
    if (existingStatus !== 'granted') {
      const { status } = await Notifications.requestPermissionsAsync();
      finalStatus = status;
    }
    
    if (finalStatus !== 'granted') {
      alert('Izin notifikasi diperlukan untuk reminder absensi!');
      return false;
    }

    return true;
  }

  /**
   * Schedule local notification
   * @param {Object} notification - Notification data from WebView
   */
  async scheduleNotification(notification) {
    try {
      const { title, body, data } = notification;

      const identifier = await Notifications.scheduleNotificationAsync({
        content: {
          title: title || 'Notifikasi',
          body: body || '',
          data: data || {},
          sound: true,
          priority: Notifications.AndroidNotificationPriority.HIGH,
          vibrate: [0, 250, 250, 250],
        },
        trigger: null, // Show immediately
      });

      console.log('Notification scheduled:', identifier);
      return identifier;
    } catch (error) {
      console.error('Error scheduling notification:', error);
      return null;
    }
  }

  /**
   * Schedule reminder notification with delay
   * @param {Object} notification - Notification data
   * @param {number} delaySeconds - Delay in seconds
   */
  async scheduleDelayedNotification(notification, delaySeconds) {
    try {
      const { title, body, data } = notification;

      const identifier = await Notifications.scheduleNotificationAsync({
        content: {
          title: title || 'Reminder',
          body: body || '',
          data: data || {},
          sound: true,
          priority: Notifications.AndroidNotificationPriority.HIGH,
        },
        trigger: {
          seconds: delaySeconds,
        },
      });

      console.log(`Notification scheduled for ${delaySeconds}s later:`, identifier);
      return identifier;
    } catch (error) {
      console.error('Error scheduling delayed notification:', error);
      return null;
    }
  }

  /**
   * Cancel all scheduled notifications
   */
  async cancelAllNotifications() {
    await Notifications.cancelAllScheduledNotificationsAsync();
    console.log('All notifications cancelled');
  }

  /**
   * Get all scheduled notifications
   */
  async getAllScheduledNotifications() {
    const notifications = await Notifications.getAllScheduledNotificationsAsync();
    return notifications;
  }
}

export default new NotificationService();
```

---

## Langkah 3: Setup WebView Component

### File: `components/AppWebView.js`

```javascript
import React, { useEffect, useRef } from 'react';
import { WebView } from 'react-native-webview';
import { Platform } from 'react-native';
import * as Notifications from 'expo-notifications';
import NotificationService from '../services/NotificationService';

const AppWebView = ({ url }) => {
  const webViewRef = useRef(null);

  useEffect(() => {
    // Request notification permissions on mount
    NotificationService.requestPermissions();

    // Setup notification response listener
    const subscription = Notifications.addNotificationResponseReceivedListener(response => {
      const data = response.notification.request.content.data;
      console.log('Notification tapped:', data);
      
      // Handle notification tap - navigate to specific screen based on data.type
      if (data.type === 'shift_swap_request') {
        // Navigate to requests page
        webViewRef.current?.injectJavaScript(`
          window.location.href = '/requested';
        `);
      } else if (data.type === 'attendance_reminder') {
        // Navigate to presensi page
        webViewRef.current?.injectJavaScript(`
          window.location.href = '/presensi';
        `);
      } else if (data.type === 'overtime_not_scheduled') {
        // Navigate to overtime page
        webViewRef.current?.injectJavaScript(`
          window.location.href = '/presensiOvTm';
        `);
      }
    });

    return () => subscription.remove();
  }, []);

  /**
   * Handle messages from WebView
   */
  const handleWebViewMessage = async (event) => {
    try {
      const message = JSON.parse(event.nativeEvent.data);
      console.log('Message from WebView:', message);

      if (message.type === 'NOTIFICATION') {
        // Schedule notification
        await NotificationService.scheduleNotification({
          title: message.title,
          body: message.body,
          data: message.data || {},
        });
      } else if (message.type === 'CANCEL_NOTIFICATIONS') {
        // Cancel all notifications
        await NotificationService.cancelAllNotifications();
      }
    } catch (error) {
      console.error('Error handling WebView message:', error);
    }
  };

  return (
    <WebView
      ref={webViewRef}
      source={{ uri: url }}
      onMessage={handleWebViewMessage}
      javaScriptEnabled={true}
      domStorageEnabled={true}
      startInLoadingState={true}
      scalesPageToFit={true}
      // Important: Set user agent to identify WebView
      userAgent={`IndoTech-App/${Platform.OS}`}
      // Allow geolocation
      geolocationEnabled={true}
      // Allow camera
      mediaCapture={true}
      allowsInlineMediaPlayback={true}
    />
  );
};

export default AppWebView;
```

---

## Langkah 4: Implementasi di App.js

### File: `App.js`

```javascript
import React, { useEffect } from 'react';
import { StyleSheet, View } from 'react-native';
import AppWebView from './components/AppWebView';
import * as Notifications from 'expo-notifications';

export default function App() {
  useEffect(() => {
    // Setup notification handler
    const subscription = Notifications.addNotificationReceivedListener(notification => {
      console.log('Notification received while app is open:', notification);
    });

    return () => subscription.remove();
  }, []);

  return (
    <View style={styles.container}>
      <AppWebView url="https://your-laravel-app-url.com/dashboard" />
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
  },
});
```

---

## Langkah 5: Testing

### 1. Test Notification dari Laravel

Di Laravel, buka browser console dan test:

```javascript
// Test kirim notifikasi
WebViewBridge.sendNotification({
  title: 'Test Notification',
  body: 'Ini adalah test notifikasi dari Laravel',
  data: { type: 'test' }
});
```

### 2. Test di React Native App

Build dan jalankan app:

```bash
# Development build
expo start

# Production build  
eas build --platform android
eas build --platform ios
```

### 3. Verify Notifications

- ✅ Notification muncul saat app terbuka
- ✅ Notification muncul saat app di background
- ✅ Tapping notification membuka halaman yang sesuai
- ✅ Sound dan vibration berfungsi

---

## Langkah 6: Handle Notification Types

Tambahkan handler untuk berbagai jenis notifikasi di `AppWebView.js`:

```javascript
const handleNotificationResponse = (response) => {
  const data = response.notification.request.content.data;
  
  switch (data.type) {
    case 'attendance_checkin_reminder':
      // Navigate to check-in page
      webViewRef.current?.injectJavaScript(`
        window.location.href = '/presensi';
      `);
      break;
      
    case 'attendance_checkout_reminder':
      // Navigate to check-out page
      webViewRef.current?.injectJavaScript(`
        window.location.href = '/presensi';
      `);
      break;
      
    case 'shift_swap_request':
      // Navigate to requests page
      webViewRef.current?.injectJavaScript(`
        window.location.href = '/requested';
      `);
      break;
      
    case 'overtime_not_scheduled':
      // Navigate to overtime page
      webViewRef.current?.injectJavaScript(`
        window.location.href = '/presensiOvTm';
      `);
      break;
      
    default:
      // Navigate to dashboard
      webViewRef.current?.injectJavaScript(`
        window.location.href = '/dashboard';
      `);
  }
};
```

---

## Troubleshooting

### Notifikasi tidak muncul?

1. **Check permissions:**
   ```javascript
   const { status } = await Notifications.getPermissionsAsync();
   console.log('Permission status:', status);
   ```

2. **Check di physical device:**
   Notifications tidak bekerja di emulator/simulator

3. **Check app.json config:**
   Pastikan plugin expo-notifications sudah terdaftar

4. **Check console log:**
   Lihat apakah message dari WebView diterima

### WebView tidak mengirim message?

1. **Check WebViewBridge loaded:**
   Buka browser console di WebView:
   ```javascript
   console.log(window.WebViewBridge);
   ```

2. **Check user agent:**
   Pastikan WebView terdeteksi sebagai IndoTech-App

3. **Test manual:**
   ```javascript
   window.ReactNativeWebView.postMessage(JSON.stringify({
     type: 'NOTIFICATION',
     title: 'Manual Test',
     body: 'Test from console'
   }));
   ```

---

## Best Practices

1. **Always request permissions early** - Di splash screen atau login
2. **Handle permission denial gracefully** - Tampilkan explanation dialog
3. **Test on real devices** - Notifications behave differently on physical devices
4. **Use meaningful notification IDs** - Untuk cancel specific notifications
5. **Don't spam notifications** - Batasi frekuensi reminder
6. **Provide notification settings** - Allow users to customize preferences

---

## Jenis Notifikasi yang Didukung

Sistem ini mendukung beberapa jenis notifikasi:

### 1. Shift Swap Request
```javascript
{
  type: 'shift_swap_request',
  title: 'Permintaan Tukar Shift',
  body: '[Nama Karyawan] mengajukan tukar shift dengan Anda...',
  data: {
    requestId: 123,
    date: '2024-01-15'
  }
}
```

### 2. Unscheduled Overtime
```javascript
{
  type: 'overtime_not_scheduled',
  title: 'Lembur Tidak Terjadwal Aktif',
  body: 'Anda memiliki lembur tidak terjadwal yang dapat dimulai kapan saja',
  data: {
    overtimeId: 456,
    date: '2024-01-15'
  }
}
```

### 3. Check-In Reminder (Coming Soon)
```javascript
{
  type: 'attendance_checkin_reminder',
  title: 'Reminder',
  body: 'Jam kerja Anda dimulai dalam 15 menit',
  data: {
    scheduleId: 789,
    minutesBefore: 15
  }
}
```

### 4. Absen Pulang Reminder (Coming Soon)
```javascript
{
  type: 'attendance_checkout_reminder',
  title: 'Reminder',
  body: 'Jangan lupa absen pulang (1/3)',
  data: {
    scheduleId: 789,
    reminderCount: 1
  }
}
```

---

## Next Steps

Setelah setup selesai, Laravel scheduler akan otomatis mengirim reminder notifications sesuai jadwal:

- ✅ Check-in reminder: 15 min sebelum sampai 15 min sesudah jam kerja
- ✅ Absen pulang reminder: Setiap 10 menit, maksimal 3x
- ✅ Shift swap request: Saat ada permintaan tukar shift
- ✅ Unscheduled overtime: Saat ada lembur tidak terjadwal aktif

---

## Support

Jika ada masalah, check:
- [Expo Notifications Docs](https://docs.expo.dev/versions/latest/sdk/notifications/)
- [React Native WebView Docs](https://github.com/react-native-webview/react-native-webview)

## Kontak

Untuk pertanyaan lebih lanjut tentang implementasi ini, silakan hubungi tim development.
