> 📚 **Dokumentasi:** [Indeks lengkap ID/EN](DOCS.md) · [English documentation index](DOCS.en.md)

# IndoTech Absensi V2


**Sistem Manajemen Kehadiran Karyawan Berbasis Web dengan WhatsApp Gateway & AI Assistant**

![Laravel](https://img.shields.io/badge/Laravel-13.0-FF2D20?style=flat-square&logo=laravel&logoColor=white)
![PHP](https://img.shields.io/badge/PHP-8.3+-777BB4?style=flat-square&logo=php&logoColor=white)
![Livewire](https://img.shields.io/badge/Livewire-4.0-FB70A9?style=flat-square)
![MySQL](https://img.shields.io/badge/MySQL-8.0-4479A1?style=flat-square&logo=mysql&logoColor=white)
![Version](https://img.shields.io/badge/Version-1.13.9-blue?style=flat-square)

---

## Deskripsi

IndoTech Absensi V2 adalah sistem manajemen kehadiran karyawan yang komprehensif dengan fitur lengkap termasuk:

- **Face Recognition** untuk absensi online
- **GPS-based Location** dengan radius validasi
- **Multi-role Access** (SuperAdmin, Admin, SDM, Kadiv, Employee)
- **Shift Management** dengan pola jadwal fleksibel
- **Request System** (izin, sakit, lembur, tukar shift, dinas luar)
- **WhatsApp Gateway** untuk notifikasi real-time
- **AI Assistant** untuk chat interaktif via WhatsApp
- **Push Notification** (Web & Mobile)
- **Export Center** (PDF, Excel)
- **Mobile App Support** (Android APK)

---

## Tech Stack

| Kategori | Teknologi |
|----------|-----------|
| **Backend** | Laravel 13.0, PHP 8.3+ |
| **Frontend** | Blade Templates, Livewire 4.0, Alpine.js, TailwindCSS |
| **UI Framework** | Material Design (Sneat Admin Template) |
| **Database** | MySQL 8.0 |
| **Queue** | Database Queue |
| **Cache** | Database Cache |
| **API** | Laravel Sanctum (REST API) |
| **Face Recognition** | face-api.js (TensorFlow.js) |
| **PDF Export** | barryvdh/laravel-dompdf |
| **Excel Import/Export** | phpoffice/phpspreadsheet |
| **Device Detection** | jenssegers/agent |
| **Mobile App** | Capacitor (Android APK) |

---

## Fitur Utama

### 1. Manajemen Kehadiran
- Absensi masuk/keluar dengan face recognition
- Validasi lokasi GPS dengan radius configurable
- Auto-detect late/early attendance
- Overtime tracking (scheduled & unscheduled)
- Missed checkout handling dengan koreksi admin

### 2. Shift & Schedule Management
- Schedule patterns (jadwal tetap per hari)
- Shift patterns (rotasi shift dengan konfigurasi)
- Shift groups (penugasan shift per tanggal)
- Support next-day shifts (shift malam)
- Holiday auto-detection

### 3. Request System
- **Schedule Request**: Izin sakit, cuti, lembur, dinas luar
- **Shift Request**: Hadir, izin, sakit, tukar shift, lembur, dinas luar
- **Shift Swap**: Tukar shift antar karyawan dengan approval dua arah
- Approval workflow dengan notifikasi WhatsApp

### 4. WhatsApp Gateway
- Notifikasi kehadiran via WhatsApp
- AI Assistant chat via WhatsApp
- Message tracking (sent, delivered, read)
- Gateway logs monitoring
- LID (LinkedIn Device) phone mapping

### 5. AI Assistant
- Chat interaktif untuk admin & employee
- Context-aware responses (database + documents)
- Configurable provider (OpenAI compatible)
- Conversation history tracking
- Export data via AI chat

### 6. Notification System
- In-app notifications
- Push notifications (Web Push API)
- WhatsApp notifications
- Notification preferences per employee
- Scheduled notification reminders

### 7. Reporting & Export
- Attendance reports dengan filter lengkap
- Details absensi per karyawan
- Export PDF & Excel
- Export Center untuk batch export

### 8. Master Data Management
- Employee management (CRUD lengkap)
- Division (Devisi) & Unit Kerja
- Lokasi Unit Kerja
- Position/Jabatan & Sub Position
- Attendance locations dengan radius
- Holiday management (auto-fetch API)
- User management (SuperAdmin only)

---

## Struktur Database

### Core Tables

| Table | Deskripsi |
|-------|-----------|
| `users` | Admin users (SuperAdmin, Admin, SDM, Kadiv) |
| `employees` | Data karyawan lengkap (HR data) |
| `attendances` | Record kehadiran (check-in/check-out) |
| `attend_locations` | Lokasi absensi (GPS coordinates) |
| `radii` | Konfigurasi radius absensi |
| `devisis` | Division/Departemen |
| `unit_kerjas` | Unit Kerja (sub-departemen) |
| `lokasi_unit_kerjas` | Lokasi spesifik per unit kerja |
| `positions` | Jabatan/Posisi |
| `position_subs` | Sub Jabatan |
| `schedules` | Jadwal kerja (template) |
| `schedule_patterns` | Pattern jadwal per hari |
| `shifts` | Shift kerja (template) |
| `shift_patterns` | Pattern shift (waktu mulai/selesai) |
| `shift_groups` | Penugasan shift per tanggal |
| `holidays` | Daftar hari libur |
| `notifications` | Notifikasi in-app |
| `device_tokens` | Token device untuk push notification |
| `notification_preferences` | Preferensi notifikasi per karyawan |
| `notification_settings` | Global notification settings |

### Request Tables

| Table | Deskripsi |
|-------|-----------|
| `schedule_requests` | Request izin/cuti/lembur (schedule-based) |
| `shift_requests` | Request izin/cuti/lembur/tukar shift (shift-based) |

### WhatsApp & AI Tables

| Table | Deskripsi |
|-------|-----------|
| `whatsapp_messages` | Tracking pesan WhatsApp |
| `wa_gateway_logs` | Log gateway WhatsApp |
| `ai_assistant_configs` | Konfigurasi AI Assistant |
| `ai_conversation_histories` | Riwayat chat WhatsApp AI |
| `admin_ai_chat_logs` | Riwayat chat AI di admin panel |
| `lid_phone_mappings` | Mapping LID ke nomor telepon |

### System Tables

| Table | Deskripsi |
|-------|-----------|
| `app_logs` | Log aktivitas aplikasi |
| `jobs` | Queue jobs |
| `cache` | Cache storage |
| `sessions` | Session storage |
| `personal_access_tokens` | API tokens (Sanctum) |

---

## Role & Permission

| Role | Deskripsi | Akses |
|------|-----------|-------|
| **SuperAdmin** | Administrator tertinggi | Semua fitur + WA Logs, App Logs, AI Settings, User Management |
| **Admin** | Administrator HR | Dashboard, Master Data, Reports, Export |
| **SDM** | Staff SDM | Dashboard, Master Data (terbatas), Reports |
| **Kadiv** | Kepala Divisi | Dashboard (divisi sendiri), Approval request |
| **Employee** | Karyawan | Dashboard, Presensi, History, Request, Profile, Schedule |

---

## Instalasi

### Prerequisites

- PHP >= 8.3
- Composer
- MySQL >= 8.0
- Node.js & NPM
- Git

### Langkah Instalasi

1. **Clone Repository**
```bash
git clone https://github.com/Ismail-IndoTech/IndoTech-Absensi_V2.git
cd IndoTech-Absensi_V2
```

2. **Install Dependencies**
```bash
composer install
npm install
```

3. **Setup Environment**
```bash
cp .env.example .env
php artisan key:generate
```

4. **Konfigurasi Database**
Edit file `.env`:
```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=indotech_absensi_online
DB_USERNAME=root
DB_PASSWORD=your_password
```

5. **Migrate & Seed Database**
```bash
php artisan migrate
php artisan db:seed
```

6. **Build Assets**
```bash
npm run build
```

7. **Setup Storage Link**
```bash
php artisan storage:link
```

8. **Run Application**
```bash
php artisan serve
```

Aplikasi akan berjalan di `http://localhost:8000`

---

## Konfigurasi Tambahan

### WhatsApp Gateway
```env
WA_GATEWAY_URL=https://your-wa-gateway.com
WA_GATEWAY_API_KEY=your_api_key
WA_WEBHOOK_SECRET=your_webhook_secret
```

### AI Assistant
```env
AI_PROVIDER=openai
AI_API_KEY=your_openai_key
AI_MODEL=gpt-4o
```

### Push Notification (Web)
```env
VAPID_PUBLIC_KEY=your_vapid_public_key
VAPID_PRIVATE_KEY=your_vapid_private_key
VAPID_SUBJECT=mailto:your@email.com
```

### Queue Worker
```bash
php artisan queue:work --tries=3
```

### Scheduled Tasks
```bash
* * * * * cd /path-to-project && php artisan schedule:run >> /dev/null 2>&1
```

Scheduled tasks yang berjalan:
- `notifications:send-attendance` - Setiap menit
- `logs:delete-old` - Setiap hari
- `holidays:fetch` - Setiap 28 Desember

---

## API Endpoints

### Authentication
| Method | Endpoint | Deskripsi |
|--------|----------|-----------|
| POST | `/api/login` | Login employee |
| POST | `/api/logout` | Logout |

### Notifications
| Method | Endpoint | Deskripsi | Auth |
|--------|----------|-----------|------|
| GET | `/api/notifications` | Get notifications | Sanctum |
| POST | `/api/notifications/{id}/read` | Mark as read | Sanctum |
| POST | `/api/notifications/{id}/respond` | Respond action | Sanctum |

### Device Tokens
| Method | Endpoint | Deskripsi | Auth |
|--------|----------|-----------|------|
| POST | `/api/device-tokens` | Register device token | Sanctum |
| DELETE | `/api/device-tokens/{token}` | Remove device token | Sanctum |

### WhatsApp & AI
| Method | Endpoint | Deskripsi |
|--------|----------|-----------|
| POST | `/api/whatsapp/webhook` | WhatsApp webhook callback |
| GET | `/api/ai-assistant/context` | Get AI context |
| POST | `/api/ai-assistant/chat` | AI chat handler |

### Export
| Method | Endpoint | Deskripsi | Auth |
|--------|----------|-----------|------|
| POST | `/api/export/admin` | Admin export | - |
| POST | `/api/export/employee` | Employee export | Employee |

### Onboarding (V2)
| Method | Endpoint | Deskripsi |
|--------|----------|-----------|
| GET | `/api/v2/onboarding` | Get onboarding structure |
| GET | `/api/v2/onboarding/{category}` | Get onboarding category |

---

## Akun Administrator (Kredensial Tidak Dipublikasikan)

| Role | Keterangan |
|------|------------|
| SuperAdmin | Kredensial tidak dipublikasikan |

> **PENTING**: Gunakan kata sandi yang kuat dan ganti setelah instalasi!

---

## Mobile App

Aplikasi mobile Android tersedia di folder `public/app-mobile/release/`:
- `GSG-Presensi-Online.apk` - Latest version
- `GSG-Presensi-Online_v1.13.9.apk` - Versioned release

Download page tersedia di `/download-app`

---

## Development

### Running in Development Mode
```bash
composer run dev
```

This runs:
- `php artisan serve`
- `php artisan queue:listen`
- `npm run dev` (Vite HMR)

### Code Style
```bash
vendor/bin/pint
```

### Testing
```bash
php artisan test
```

---

## Project Structure

```
IndoTech-Absensi_V2/
├── app/
│   ├── Console/Commands/          # Artisan commands
│   ├── Http/
│   │   ├── Controllers/           # Controllers (Admin, User, API)
│   │   └── Middleware/            # Custom middleware
│   ├── Imports/                   # Excel import handlers
│   ├── Livewire/                  # Livewire components
│   │   ├── Admin/                 # Admin dashboard components
│   │   └── User/                  # Employee dashboard components
│   ├── Models/                    # Eloquent models
│   ├── Services/                  # Service classes
│   ├── Traits/                    # Reusable traits
│   └── View/Components/           # Blade components
├── config/                        # Configuration files
├── database/
│   ├── factories/                 # Model factories
│   ├── migrations/                # Database migrations
│   └── seeders/                   # Database seeders
├── public/
│   ├── app-mobile/                # Mobile app landing page & APK
│   ├── auth/                      # Auth assets
│   ├── models/                    # Face recognition models
│   └── user/                      # User panel assets
├── resources/
│   ├── css/                       # CSS files
│   ├── js/                        # JavaScript files
│   └── views/                     # Blade templates
│       ├── admin/                 # Admin views
│       ├── user/                  # Employee views
│       └── components/            # Layout components
├── routes/
│   ├── web.php                    # Web routes
│   ├── api.php                    # API routes
│   └── console.php                # Scheduled tasks
├── storage/                       # Storage (logs, cache, etc.)
└── tests/                         # Test files
```

---

## 💖 Kontribusi & Donasi

Jika aplikasi ini mempermudah operasional IT, DevOps, dan pengelolaan server di perusahaan atau proyek pribadi Anda, dukung pengembangannya dengan menjadi kontributor atau **Donatur Spesial**.  
*(Donatur dan Sponsor resmi akan kami cantumkan logo perusahaannya di repositori ini).*

Dukung pemeliharaan berkelanjutan, riset modul bypass panel baru, dan pembaruan sistem dengan berdonasi melalui:

<div align="center">

**DANA — 08121600058** (a.n. Mochamad Ismail Soleh)

<img src="docs/qris.jpg" alt="QRIS DANA IndoTech" width="280">

*Pindai dengan aplikasi DANA, BCA, Mandiri, GoPay, OVO, atau aplikasi pembayaran QRIS apa pun di Indonesia.*

</div>

Semua dana dialokasikan untuk biaya infrastruktur riset server, lisensi alat pengujian, dan pengembangan modul open-source IndoTech. Jika Anda atau institusi Anda ingin menjadi mitra resmi atau sponsor, silakan hubungi kami. Terima kasih atas dukungan Anda!

---

## License

MIT License

---

## Author

**Ismail-IndoTech**

- GitHub: https://github.com/Ismail-IndoTech

---

## Support

Untuk pertanyaan atau dukungan, silakan buat issue di repository atau hubungi tim development.