# GSG Attendance Documentation

Technical documentation for the IndoTech attendance system.

## 📱 React Native / Mobile App

### [Push Notification Integration Tutorial](./REACT_NATIVE_NOTIFICATION_TUTORIAL.en.md)

A complete tutorial for integrating push notifications from the Laravel web app into a React Native/Expo mobile app using a WebView bridge.

**Covers:**
- expo-notifications setup
- WebView configuration
- Handle notification types
- Testing & troubleshooting

---

## 🔔 Notification System

This application uses a **WebView Bridge** for communication between Laravel and React Native:

```
Laravel -> WebView Bridge -> React Native -> Expo Notifications
```

### Supported Notification Types

1. **Shift Swap Request** - Notification when there is a shift swap request
2. **Unscheduled Overtime** - Unscheduled overtime notification
3. **Check-in Reminder** *(planned)* - Reminder 15 minutes before/after working hours
4. **Check-out Reminder** *(planned)* - Reminder 10, 20, 30 minutes after working hours end

---

## 📂 Documentation Structure

```
docs/
|-- README.md                                    # This file
+-- REACT_NATIVE_NOTIFICATION_TUTORIAL.md       # React Native tutorial
```

---

## 🚀 Quick Start

### For React Native Developers

1. Read [REACT_NATIVE_NOTIFICATION_TUTORIAL.en.md](./REACT_NATIVE_NOTIFICATION_TUTORIAL.en.md)
2. Install required packages:
   ```bash
   npx expo install expo-notifications expo-device react-native-webview
   ```
3. Follow the step-by-step tutorial
4. Test on a physical device

### For Backend Developers

The WebView bridge is already implemented in:
- `public/user/assets/js/webview-bridge.js`
- `resources/views/components/layout/user.blade.php`
- `resources/views/user/dashboard.blade.php`

To add a new notification:
```javascript
if (window.WebViewBridge && window.WebViewBridge.isWebView()) {
    window.WebViewBridge.sendNotification({
        title: 'Judul Notifikasi',
        body: 'Pesan notifikasi',
        data: {
            type: 'notification_type',
            // custom data
        },
        priority: 'high' // or 'default'
    });
}
```

---

## 🔧 Development

### Testing WebView Bridge

In the browser console:
```javascript
// Check if bridge loaded
console.log(window.WebViewBridge);

// Check if in WebView (will be false in the browser)
console.log(WebViewBridge.isWebView());

// Test send notification (only works in WebView)
WebViewBridge.sendNotification({
    title: 'Test',
    body: 'Test notification',
    data: { type: 'test' }
});
```

---

## 📝 Changelog

### 2025-12-30
- ✅ Implemented WebView Bridge
- ✅ Added shift swap notifications
- ✅ Added unscheduled overtime notifications
- ✅ Created React Native integration tutorial

---

## 📞 Support

For questions or issues, contact the IndoTech development team.
