> 📚 **Documentation:** [Full ID/EN index](DOCS.md) · [English index](DOCS.en.md)

# IndoTech Absensi V2


**Web-Based Employee Attendance Management System with WhatsApp Gateway & AI Assistant**

![Laravel](https://img.shields.io/badge/Laravel-13.0-FF2D20?style=flat-square&logo=laravel&logoColor=white)
![PHP](https://img.shields.io/badge/PHP-8.3+-777BB4?style=flat-square&logo=php&logoColor=white)
![Livewire](https://img.shields.io/badge/Livewire-4.0-FB70A9?style=flat-square)
![MySQL](https://img.shields.io/badge/MySQL-8.0-4479A1?style=flat-square&logo=mysql&logoColor=white)
![Version](https://img.shields.io/badge/Version-1.13.9-blue?style=flat-square)

---

## Description

IndoTech Absensi V2 is a comprehensive employee attendance management system with complete features including:

- **Face Recognition** for online attendance
- **GPS-based Location** with validation radius
- **Multi-role Access** (SuperAdmin, Admin, SDM, Kadiv, Employee)
- **Shift Management** with flexible schedule patterns
- **Request System** (leave, sick, overtime, shift swap, official outside duty)
- **WhatsApp Gateway** for real-time notifications
- **AI Assistant** for interactive chat via WhatsApp
- **Push Notification** (Web & Mobile)
- **Export Center** (PDF, Excel)
- **Mobile App Support** (Android APK)

---

## Tech Stack

| Category | Technology |
|----------|------------|
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

## Main Features

### 1. Attendance Management
- Check-in/check-out with face recognition
- GPS location validation with configurable radius
- Auto-detect late/early attendance
- Overtime tracking (scheduled & unscheduled)
- Missed checkout handling with admin correction

### 2. Shift & Schedule Management
- Schedule patterns (fixed schedule per day)
- Shift patterns (shift rotation with configuration)
- Shift groups (shift assignment per date)
- Support next-day shifts (night shift)
- Holiday auto-detection

### 3. Request System
- **Schedule Request**: Sick leave, vacation, overtime, official outside duty
- **Shift Request**: Present, leave, sick, shift swap, overtime, official outside duty
- **Shift Swap**: Shift swap between employees with two-way approval
- Approval workflow with WhatsApp notifications

### 4. WhatsApp Gateway
- Attendance notifications via WhatsApp
- AI Assistant chat via WhatsApp
- Message tracking (sent, delivered, read)
- Gateway logs monitoring
- LID (LinkedIn Device) phone mapping

### 5. AI Assistant
- Interactive chat for admin & employee
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
- Attendance reports with complete filters
- Attendance details per employee
- PDF & Excel export
- Export Center for batch export

### 8. Master Data Management
- Employee management (complete CRUD)
- Division (Devisi) & Work Unit
- Work Unit Location
- Position/Job Title & Sub Position
- Attendance locations with radius
- Holiday management (auto-fetch API)
- User management (SuperAdmin only)

---

## Database Structure

### Core Tables

| Table | Description |
|-------|-------------|
| `users` | Admin users (SuperAdmin, Admin, SDM, Kadiv) |
| `employees` | Complete employee data (HR data) |
| `attendances` | Attendance records (check-in/check-out) |
| `attend_locations` | Attendance locations (GPS coordinates) |
| `radii` | Attendance radius configuration |
| `devisis` | Division/Department |
| `unit_kerjas` | Work Unit (sub-department) |
| `lokasi_unit_kerjas` | Specific location per work unit |
| `positions` | Position/Job Title |
| `position_subs` | Sub Position |
| `schedules` | Work schedule (template) |
| `schedule_patterns` | Schedule pattern per day |
| `shifts` | Work shift (template) |
| `shift_patterns` | Shift pattern (start/end time) |
| `shift_groups` | Shift assignment per date |
| `holidays` | Holiday list |
| `notifications` | In-app notifications |
| `device_tokens` | Device tokens for push notifications |
| `notification_preferences` | Notification preferences per employee |
| `notification_settings` | Global notification settings |

### Request Tables

| Table | Description |
|-------|-------------|
| `schedule_requests` | Leave/sick/overtime requests (schedule-based) |
| `shift_requests` | Leave/sick/overtime/shift swap requests (shift-based) |

### WhatsApp & AI Tables

| Table | Description |
|-------|-------------|
| `whatsapp_messages` | WhatsApp message tracking |
| `wa_gateway_logs` | WhatsApp gateway logs |
| `ai_assistant_configs` | AI Assistant configuration |
| `ai_conversation_histories` | WhatsApp AI chat history |
| `admin_ai_chat_logs` | AI chat history in admin panel |
| `lid_phone_mappings` | LID to phone number mapping |

### System Tables

| Table | Description |
|-------|-------------|
| `app_logs` | Application activity logs |
| `jobs` | Queue jobs |
| `cache` | Cache storage |
| `sessions` | Session storage |
| `personal_access_tokens` | API tokens (Sanctum) |

---

## Role & Permission

| Role | Description | Access |
|------|-------------|--------|
| **SuperAdmin** | Highest administrator | All features + WA Logs, App Logs, AI Settings, User Management |
| **Admin** | HR administrator | Dashboard, Master Data, Reports, Export |
| **SDM** | HR staff | Dashboard, Master Data (limited), Reports |
| **Kadiv** | Division Head | Dashboard (own division), Request approval |
| **Employee** | Employee | Dashboard, Attendance, History, Request, Profile, Schedule |

---

## Installation

### Prerequisites

- PHP >= 8.3
- Composer
- MySQL >= 8.0
- Node.js & NPM
- Git

### Installation Steps

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

4. **Database Configuration**
Edit the `.env` file:
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

The application will run at `http://localhost:8000`

---

## Additional Configuration

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

Scheduled tasks that run:
- `notifications:send-attendance` - Every minute
- `logs:delete-old` - Every day
- `holidays:fetch` - Every December 28

---

## API Endpoints

### Authentication
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/login` | Employee login |
| POST | `/api/logout` | Logout |

### Notifications
| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| GET | `/api/notifications` | Get notifications | Sanctum |
| POST | `/api/notifications/{id}/read` | Mark as read | Sanctum |
| POST | `/api/notifications/{id}/respond` | Respond action | Sanctum |

### Device Tokens
| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| POST | `/api/device-tokens` | Register device token | Sanctum |
| DELETE | `/api/device-tokens/{token}` | Remove device token | Sanctum |

### WhatsApp & AI
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/whatsapp/webhook` | WhatsApp webhook callback |
| GET | `/api/ai-assistant/context` | Get AI context |
| POST | `/api/ai-assistant/chat` | AI chat handler |

### Export
| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| POST | `/api/export/admin` | Admin export | - |
| POST | `/api/export/employee` | Employee export | Employee |

### Onboarding (V2)
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/v2/onboarding` | Get onboarding structure |
| GET | `/api/v2/onboarding/{category}` | Get onboarding category |

---

## Administrator Account (Credentials Not Published)

| Role | Description |
|------|-------------|
| SuperAdmin | Credentials not published |

> **IMPORTANT**: Use a strong password and change it after installation!

---

## Mobile App

The Android mobile application is available in the `public/app-mobile/release/` folder:
- `GSG-Presensi-Online.apk` - Latest version
- `GSG-Presensi-Online_v1.13.9.apk` - Versioned release

The download page is available at `/download-app`

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
|-- app/
|   |-- Console/Commands/          # Artisan commands
|   |-- Http/
|   |   |-- Controllers/           # Controllers (Admin, User, API)
|   |   +-- Middleware/            # Custom middleware
|   |-- Imports/                   # Excel import handlers
|   |-- Livewire/                  # Livewire components
|   |   |-- Admin/                 # Admin dashboard components
|   |   +-- User/                  # Employee dashboard components
|   |-- Models/                    # Eloquent models
|   |-- Services/                  # Service classes
|   |-- Traits/                    # Reusable traits
|   +-- View/Components/           # Blade components
|-- config/                        # Configuration files
|-- database/
|   |-- factories/                 # Model factories
|   |-- migrations/                # Database migrations
|   +-- seeders/                   # Database seeders
|-- public/
|   |-- app-mobile/                # Mobile app landing page & APK
|   |-- auth/                      # Auth assets
|   |-- models/                    # Face recognition models
|   +-- user/                      # User panel assets
|-- resources/
|   |-- css/                       # CSS files
|   |-- js/                        # JavaScript files
|   +-- views/                     # Blade templates
|       |-- admin/                 # Admin views
|       |-- user/                  # Employee views
|       +-- components/            # Layout components
|-- routes/
|   |-- web.php                    # Web routes
|   |-- api.php                    # API routes
|   +-- console.php                # Scheduled tasks
|-- storage/                       # Storage (logs, cache, etc.)
+-- tests/                         # Test files
```

---

## 💖 Contribution & Donation

If this application makes IT operations, DevOps, and server management easier in your company or personal project, support its development by becoming a contributor or a **Special Donor**.  
*(We will list the company logos of official donors and sponsors in this repository).*

Support ongoing maintenance, research into new panel bypass modules, and system updates by donating via:

<div align="center">

**DANA — 08121600058** (on behalf of Mochamad Ismail Soleh)

<img src="docs/qris.jpg" alt="QRIS DANA IndoTech" width="280">

*Scan with the DANA, BCA, Mandiri, GoPay, OVO, or any QRIS payment application in Indonesia.*

</div>

All funds are allocated for research server infrastructure costs, testing tool licenses, and the development of IndoTech open-source modules. If you or your institution would like to become an official partner or sponsor, please contact us. Thank you for your support!

---

## License

MIT License

---

## Author

**Ismail-IndoTech**

- GitHub: https://github.com/Ismail-IndoTech

---

## Support

For questions or support, please create an issue in the repository or contact the development team.