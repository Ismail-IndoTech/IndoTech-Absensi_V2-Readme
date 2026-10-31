# Dokumentasi Presensi GSG

Dokumentasi teknis untuk sistem presensi IndoTech.

## 📱 React Native / Mobile App

### [Tutorial Integrasi Notifikasi Push](./REACT_NATIVE_NOTIFICATION_TUTORIAL.md)

Tutorial lengkap untuk mengintegrasikan push notifications dari Laravel web app ke React Native/Expo mobile app menggunakan WebView bridge.

**Mencakup:**
- Setup expo-notifications
- Konfigurasi WebView
- Handle notification types
- Testing & troubleshooting

---

## 🔔 Sistem Notifikasi

Aplikasi ini menggunakan **WebView Bridge** untuk komunikasi antara Laravel dan React Native:

```
Laravel → WebView Bridge → React Native → Expo Notifications
```

### Jenis Notifikasi yang Didukung

1. **Shift Swap Request** - Notifikasi saat ada permintaan tukar shift
2. **Unscheduled Overtime** - Notifikasi lembur tidak terjadwal
3. **Check-in Reminder** *(planned)* - Reminder 15 menit sebelum/sesudah jam kerja
4. **Absen Pulang Reminder** *(planned)* - Reminder 10, 20, 30 menit setelah jam pulang

---

## 📂 Struktur Dokumentasi

```
docs/
├── README.md                                    # File ini
└── REACT_NATIVE_NOTIFICATION_TUTORIAL.md       # Tutorial React Native
```

---

## 🚀 Quick Start

### Untuk Developer React Native

1. Baca [REACT_NATIVE_NOTIFICATION_TUTORIAL.md](./REACT_NATIVE_NOTIFICATION_TUTORIAL.md)
2. Install required packages:
   ```bash
   npx expo install expo-notifications expo-device react-native-webview
   ```
3. Follow step-by-step dalam tutorial
4. Test di physical device

### Untuk Backend Developer

WebView bridge sudah terimplementasi di:
- `public/user/assets/js/webview-bridge.js`
- `resources/views/components/layout/user.blade.php`
- `resources/views/user/dashboard.blade.php`

Untuk menambah notifikasi baru:
```javascript
if (window.WebViewBridge && window.WebViewBridge.isWebView()) {
    window.WebViewBridge.sendNotification({
        title: 'Judul Notifikasi',
        body: 'Pesan notifikasi',
        data: {
            type: 'notification_type',
            // custom data
        },
        priority: 'high' // atau 'default'
    });
}
```

---

## 🔧 Development

### Testing WebView Bridge

Di browser console:
```javascript
// Check if bridge loaded
console.log(window.WebViewBridge);

// Check if in WebView (akan false di browser)
console.log(WebViewBridge.isWebView());

// Test send notification (hanya bekerja di WebView)
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

Untuk pertanyaan atau issues, hubungi tim development IndoTech.
