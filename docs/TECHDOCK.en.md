# TechDock - IndoTech Absensi V2

**Technical Documentation & Knowledge Base**

Version: 1.13.9 | Last Updated: May 17, 2026

---

## Table of Contents

1. [System Architecture](#1-system-architecture)
2. [Application Workflow](#2-application-workflow)
3. [Database Schema](#3-database-schema)
4. [Authentication & Authorization](#4-authentication--authorization)
5. [API Reference](#5-api-reference)
6. [User Guide](#6-user-guide)
7. [WhatsApp Gateway Integration](#7-whatsapp-gateway-integration)
8. [AI Assistant System](#8-ai-assistant-system)
9. [Notification System](#9-notification-system)
10. [Attendance System](#10-attendance-system)
11. [Shift & Schedule Management](#11-shift--schedule-management)
12. [Request & Approval Workflow](#12-request--approval-workflow)
13. [Export & Reporting](#13-export--reporting)
14. [Deployment Guide](#14-deployment-guide)
15. [Troubleshooting](#15-troubleshooting)

---

## 1. System Architecture

### 1.1 Overview

```
+-----------------------------------------------------------------+
|                        CLIENT LAYER                              |
+-----------------+------------------+----------------------------+
|   Web Browser   |   Mobile App     |   WhatsApp Client          |
|   (Admin/User)  |   (Android APK)  |   (Employees via WA)       |
+--------+--------+--------+---------+-------------+--------------+
         |                 |                       |
         v                 v                       v
+-----------------------------------------------------------------+
|                     APPLICATION LAYER                            |
+-----------------------------------------------------------------+
|                    Laravel 13.0 Application                      |
|  +-------------+  +--------------+  +------------------------+  |
|  | Web Routes  |  |  API Routes  |  |  Scheduled Tasks       |  |
|  | (Blade +    |  |  (Sanctum)   |  |  (Queue + Cron)        |  |
|  |  Livewire)  |  |              |  |                        |  |
|  +-------------+  +--------------+  +------------------------+  |
|  +-------------------------------------------------------------+ |
|  |                    Service Layer                             | |
|  |  WhatsAppGateway | AI Assistant | Push Notification | Export | |
|  +-------------------------------------------------------------+ |
+-----------------------------------------------------------------+
         |                 |                       |
         v                 v                       v
+-----------------------------------------------------------------+
|                       DATA LAYER                                 |
+-----------------+------------------+----------------------------+
|   MySQL 8.0     |   Cache/Session  |   File Storage             |
|   (35+ tables)  |   (Database)     |   (Local Disk)             |
+-----------------+------------------+----------------------------+
```

### 1.2 Technology Stack

| Layer | Technology | Purpose |
|-------|------------|---------|
| **Backend Framework** | Laravel 13.0 | MVC framework, routing, ORM |
| **PHP Version** | 8.3+ | Runtime environment |
| **Frontend (Web)** | Blade + Livewire 4.0 + Alpine.js | Reactive UI components |
| **CSS Framework** | TailwindCSS + Material Design (Sneat) | Styling & UI components |
| **Database** | MySQL 8.0 | Primary data storage |
| **Queue System** | Database Queue | Background job processing |
| **Cache** | Database Cache | Performance optimization |
| **API Auth** | Laravel Sanctum | Token-based API authentication |
| **Face Recognition** | face-api.js (TensorFlow.js) | Client-side face detection |
| **PDF Generation** | barryvdh/laravel-dompdf | PDF export |
| **Excel Processing** | phpoffice/phpspreadsheet | Import/Export Excel |
| **Push Notifications** | Web Push API (VAPID) | Browser notifications |
| **Mobile App** | Capacitor (Android) | Native Android APK |

### 1.3 Directory Structure

```
app/
|-- Console/Commands/      # Artisan commands (scheduled tasks)
|-- Helpers/               # Helper functions (IpHelper)
|-- Http/
|   |-- Controllers/       # Request handlers (Web + API)
|   +-- Middleware/        # Request filtering & auth
|-- Imports/               # Excel import handlers
|-- Livewire/              # Reactive components (Admin + User)
|-- Models/                # Eloquent ORM models
|-- Services/              # Business logic services
|-- Traits/                # Reusable code traits
+-- View/Components/       # Blade components

database/
|-- factories/             # Test data factories
|-- migrations/            # Database schema (50 migrations)
+-- seeders/               # Initial data seeders

resources/views/
|-- admin/                 # Admin panel views
|-- user/                  # Employee panel views
|-- auth/                  # Authentication views
|-- components/            # Layout components
|-- exports/               # PDF templates
+-- livewire/              # Livewire view templates

routes/
|-- web.php                # Web application routes
|-- api.php                # REST API routes
+-- console.php            # Scheduled task definitions
```

---

## 2. Application Workflow

### 2.1 User Registration & Onboarding

```
1. Admin creates employee account
   +-- Admin Panel > Master Data > Employee > Add New
   +-- Required: badge, full_name, phone, email, devisi, position

2. Employee receives credentials
   +-- Phone number as username
   +-- Default password (set by admin)

3. First login
   +-- Employee logs in via web or mobile app
   +-- System validates credentials against employees table
   +-- Session created with role-based access

4. Device binding (optional)
   +-- Mobile app registers device token
   +-- Device UUID stored for security
```

### 2.2 Daily Attendance Flow

```
Morning Check-in:
+-------------+     +--------------+     +---------------+
| Employee    |---->| Face Detect  |---->| GPS Location  |
| Opens App   |     | (face-api.js)|     | Validation    |
+-------------+     +--------------+     +-------+-------+
                                                  |
                                    +-------------v-------------+
                                    |     Validation Check      |
                                    |  1. Face detected?        |
                                    |  2. Within radius?        |
                                    |  3. Within time window?   |
                                    |  4. Not holiday?          |
                                    +-------------+-------------+
                                                  |
                                    +-------------v-------------+
                                    |     Record Attendance     |
                                    |  - Timestamp              |
                                    |  - Location (in)          |
                                    |  - Photo (pict_in)        |
                                    |  - Status (ontime/late)   |
                                    +-------------+-------------+
                                                  |
                                    +-------------v-------------+
                                    |   Send Notification       |
                                    |  - WhatsApp (if enabled)  |
                                    |  - Push notification      |
                                    |  - In-app notification    |
                                    +---------------------------+

Evening Check-out:
Same flow with location_out, pict_out, early_time calculation
```

### 2.3 Request Approval Flow

```
Employee submits request:
+-------------+     +--------------+     +---------------+
| Employee    |---->| Request      |---->| Notification  |
| Creates     |     | Created      |     | Sent to Admin |
| Request     |     | (pending)    |     | / Kadiv       |
+-------------+     +--------------+     +-------+-------+
                                                  |
                                    +-------------v-------------+
                                    |   Admin/Kadiv Reviews     |
                                    |   - View request details  |
                                    |   - Check attachments     |
                                    |   - Approve or Reject     |
                                    +-------------+-------------+
                                                  |
                          +-----------------------+---------------+
                          v                       v               v
                    +-----------+          +-----------+   +-----------+
                    | Approved  |          | Rejected  |   | Pending   |
                    | Status:   |          | Status:   |   | Awaiting  |
                    | approve   |          | reject    |   | response  |
                    +-----+-----+          +-----+-----+   +-----------+
                          |                      |
                          v                      v
                    +----------------------------------+
                    |  Notification to Employee        |
                    |  - WhatsApp message              |
                    |  - In-app notification           |
                    |  - Push notification             |
                    +----------------------------------+
```

### 2.4 Shift Swap Flow

```
Employee A wants to swap shift with Employee B:

1. Employee A creates shift request
   +-- type: change_shift
   +-- target_employee_id: Employee B
   +-- target_start_date, target_end_date
   +-- target_start_time, target_end_time

2. System creates reciprocal request
   +-- swap_request_id links both requests

3. Employee B receives notification
   +-- WhatsApp + in-app notification
   +-- Can accept or decline

4. If Employee B accepts:
   +-- target_accepted = true
   +-- Forwarded to Admin/Kadiv for final approval

5. Admin approves:
   +-- Both shift assignments updated
   +-- Both employees notified
```

---

## 3. Database Schema

### 3.1 Entity Relationship Overview

```
+--------------+       +--------------+       +--------------+
|    users     |       |   devisis    |       |  positions   |
+--------------+       +--------------+       +--------------+
| id           |       | id           |       | id           |
| name         |       | name         |       | name         |
| email        |       | ket          |       | ket          |
| password     |       | kadiv_id ----+------>|              |
| role         |       +------+-------+       +------+-------+
| status       |              |                      |
+--------------+              |                      |
                              v                      v
                        +----------------------------------+
                        |          employees               |
                        +----------------------------------+
                        | id, badge, full_name, phone      |
                        | email, password, status, uuid    |
                        | devisi_id -----------------------+--> devisis
                        | position_id ---------------------+--> positions
                        | att_loc_id ----------------------+--> attend_locations
                        | schedule_id ---------------------+--> schedules
                        | shift_id ------------------------+--> shifts
                        | unit_kerja_id -------------------+--> unit_kerjas
                        | lokasi_unit_kerja_id ------------+--> lokasi_unit_kerjas
                        | place_birth, birth_date, gender  |
                        | nik_ktp, address, join_date      |
                        | accuracy_opts, radius_opts       |
                        +--------------+-------------------+
                                       |
                    +------------------+------------------+
                    v                  v                  v
            +--------------+  +--------------+  +--------------+
            | attendances  |  |schedule_reqs |  | shift_reqs   |
            +--------------+  +--------------+  +--------------+
            | id           |  | id           |  | id           |
            | employee_id  |  | employee_id  |  | employee_id  |
            | schedule_id  |  | type         |  | type         |
            | shift_id     |  | start_date   |  | start_date   |
            | start_attd_  |  | end_date     |  | end_date     |
            |   date       |  | status       |  | status       |
            | attd_in_time |  | reason, ket  |  | reason, ket  |
            | attd_out_time|  | image        |  | image        |
            | location_in  |  | approved_by  |  | approved_by  |
            | location_out |  | rejected_by  |  | rejected_by  |
            | pict_in/out  |  +--------------+  | target_emp_id|
            | status_in/out|                    | swap_req_id  |
            | type         |                    +--------------+
            +--------------+
```

### 3.2 Core Tables

#### users
Admin panel users (SuperAdmin, Admin, SDM, Kadiv)

| Column | Type | Description |
|--------|------|-------------|
| id | BIGINT | Primary key |
| name | VARCHAR | Full name |
| email | VARCHAR | Unique email |
| password | VARCHAR | Hashed password |
| role | ENUM | 'SuperAdmin', 'Admin', 'SDM', 'Kadiv' |
| status | INT | 0 = Active, 1 = Inactive |
| created_at | TIMESTAMP | Record creation time |
| updated_at | TIMESTAMP | Last update time |

#### employees
Employee master data

| Column | Type | Description |
|--------|------|-------------|
| id | BIGINT | Primary key |
| badge | VARCHAR | Employee badge number (unique) |
| full_name | VARCHAR | Full name |
| nick_name | VARCHAR | Nickname (optional) |
| phone | VARCHAR | Phone number (unique) |
| email | VARCHAR | Email (unique) |
| password | VARCHAR | Hashed password |
| status | INT | 0 = Active, 1 = Inactive, 2 = Retired |
| uuid | VARCHAR | Device UUID (for binding) |
| devisi_id | BIGINT FK | References devisis |
| unit_kerja_id | BIGINT FK | References unit_kerjas |
| lokasi_unit_kerja_id | BIGINT FK | References lokasi_unit_kerjas |
| position_id | BIGINT FK | References positions |
| att_loc_id | BIGINT FK | Default attendance location |
| schedule_id | BIGINT FK | Assigned schedule |
| shift_id | BIGINT FK | Assigned shift |
| place_birth | VARCHAR | Birth place |
| birth_date | DATE | Birth date |
| gender | ENUM | 'L' (Male), 'P' (Female) |
| nik_ktp | VARCHAR(16) | National ID number |
| address | TEXT | Address |
| join_date | DATE | Join date |
| transfer_date | DATE | Transfer date |
| religion | ENUM | Religion options |
| image | VARCHAR | Profile photo path |
| accuracy_opts | INT | Custom accuracy override |
| radius_opts | INT | Custom radius override |
| is_approval | BOOLEAN | Can approve requests |

#### attendances
Attendance records (check-in/check-out)

| Column | Type | Description |
|--------|------|-------------|
| id | BIGINT | Primary key |
| employee_id | BIGINT FK | References employees |
| schedule_id | BIGINT FK | References schedules |
| schedule_pattern_id | BIGINT FK | References schedule_patterns |
| shift_id | BIGINT FK | References shifts |
| shift_pattern_id | BIGINT FK | References shift_patterns |
| sched_req_id | BIGINT FK | References schedule_requests |
| shift_req_id | BIGINT FK | References shift_requests |
| start_attd_date | DATE | Attendance date |
| end_attd_date | DATE | End date (for overnight) |
| attd_in_time | TIME | Check-in time |
| attd_out_time | TIME | Check-out time |
| late_time | TIME | Late duration |
| early_time | TIME | Early departure duration |
| location_in | VARCHAR | Check-in location name |
| location_out | VARCHAR | Check-out location name |
| pict_in | VARCHAR | Check-in photo path |
| pict_out | VARCHAR | Check-out photo path |
| type | ENUM | 'present', 'requested', 'changeSched', 'overtime', 'overtimeNotSched' |
| status_in | ENUM | 'ontime', 'late' |
| status_out | ENUM | 'ontime', 'early' |

#### attend_locations
Attendance location definitions

| Column | Type | Description |
|--------|------|-------------|
| id | BIGINT | Primary key |
| name | VARCHAR | Location name |
| latitude | VARCHAR | GPS latitude |
| longitude | VARCHAR | GPS longitude |
| radius_id | BIGINT FK | References radii |
| accuracy | INT | GPS accuracy threshold (meters) |
| ket | TEXT | Description/notes |

#### radii
Radius configuration

| Column | Type | Description |
|--------|------|-------------|
| id | BIGINT | Primary key |
| name | VARCHAR | Radius name |
| radius | UNSIGNED INT | Radius in meters |
| ket | TEXT | Description |

### 3.3 Schedule & Shift Tables

#### schedules
Schedule templates

| Column | Type | Description |
|--------|------|-------------|
| id | BIGINT | Primary key |
| name | VARCHAR | Schedule name |
| ket | TEXT | Description |

#### schedule_patterns
Daily schedule patterns

| Column | Type | Description |
|--------|------|-------------|
| id | BIGINT | Primary key |
| schedule_id | BIGINT FK | References schedules |
| name | VARCHAR | Pattern name |
| day | ENUM | 'monday' - 'sunday' |
| start_time | TIME | Start time |
| end_time | TIME | End time |
| ket | TEXT | Description |

#### shifts
Shift templates

| Column | Type | Description |
|--------|------|-------------|
| id | BIGINT | Primary key |
| name | VARCHAR | Shift name |
| ket | TEXT | Description |
| shift_pattern_ids | JSON | Array of shift pattern IDs |

#### shift_patterns
Shift time configurations

| Column | Type | Description |
|--------|------|-------------|
| id | BIGINT | Primary key |
| name | VARCHAR | Pattern name |
| start_time | TIME | Shift start time |
| end_time | TIME | Shift end time |
| next_day | BOOLEAN | Extends to next day |
| can_change | BOOLEAN | Allow shift change requests |
| ignore_holiday | BOOLEAN | Active on holidays |

#### shift_groups
Daily shift assignments

| Column | Type | Description |
|--------|------|-------------|
| id | BIGINT | Primary key |
| date | DATE | Assignment date |
| changed | BOOLEAN | Shift was changed |
| shift_id | BIGINT FK | References shifts |
| shift_pattern_id | BIGINT FK | References shift_patterns |

### 3.4 Organizational Tables

#### devisis
Divisions/Departments

| Column | Type | Description |
|--------|------|-------------|
| id | BIGINT | Primary key |
| name | VARCHAR | Division name |
| ket | TEXT | Description |
| kadiv_id | BIGINT FK | Division head (employee) |

#### unit_kerjas
Work units (sub-divisions)

| Column | Type | Description |
|--------|------|-------------|
| id | BIGINT | Primary key |
| devisi_id | BIGINT FK | Parent division |
| name | VARCHAR | Unit name |
| ket | TEXT | Description |

#### lokasi_unit_kerjas
Work unit locations

| Column | Type | Description |
|--------|------|-------------|
| id | BIGINT | Primary key |
| unit_kerja_id | BIGINT FK | Parent work unit |
| name | VARCHAR | Location name |
| ket | TEXT | Description |

#### positions
Job positions

| Column | Type | Description |
|--------|------|-------------|
| id | BIGINT | Primary key |
| name | VARCHAR | Position name |
| ket | TEXT | Description |

#### position_subs
Sub-positions

| Column | Type | Description |
|--------|------|-------------|
| id | BIGINT | Primary key |
| position_id | BIGINT FK | Parent position |
| name | VARCHAR | Sub-position name |
| ket | TEXT | Description |

### 3.5 Request Tables

#### schedule_requests
Schedule-based requests (leave, vacation, overtime)

| Column | Type | Description |
|--------|------|-------------|
| id | BIGINT | Primary key |
| employee_id | BIGINT FK | Requesting employee |
| devisi_id | BIGINT FK | Employee's division |
| type | ENUM | 'sick', 'leave', 'overtime', 'overtimeNotSched', 'dinas_luar' |
| start_date | DATE | Start date |
| end_date | DATE | End date |
| start_time | TIME | Start time (nullable) |
| end_time | TIME | End time (nullable) |
| next_day | BOOLEAN | Overnight shift |
| status | ENUM | 'pending', 'approve', 'reject' |
| reason | TEXT | Approval reason |
| ket | TEXT | Employee's note |
| image | VARCHAR | Attachment photo |
| is_out_location | BOOLEAN | Outside location request |
| approved_by | BIGINT | Approver ID |
| rejected_by | BIGINT | Rejector ID |

#### shift_requests
Shift-based requests (includes shift swap)

| Column | Type | Description |
|--------|------|-------------|
| id | BIGINT | Primary key |
| employee_id | BIGINT FK | Requesting employee |
| devisi_id | BIGINT FK | Employee's division |
| type | ENUM | 'present', 'leave', 'sick', 'change_shift', 'overtime', 'overtimeNotSched', 'dinas_luar' |
| start_date | DATE | Start date |
| end_date | DATE | End date |
| start_time | TIME | Start time |
| end_time | TIME | End time |
| next_day | BOOLEAN | Overnight shift |
| status | ENUM | 'pending', 'approve', 'reject' |
| reason | TEXT | Approval reason |
| ket | TEXT | Employee's note |
| image | VARCHAR | Attachment photo |
| is_out_location | BOOLEAN | Outside location request |
| target_employee_id | BIGINT FK | Target for shift swap |
| target_shift_pattern_id | BIGINT FK | Target's shift pattern |
| target_start_date | DATE | Target's start date |
| target_end_date | DATE | Target's end date |
| target_start_time | TIME | Target's start time |
| target_end_time | TIME | Target's end time |
| target_accepted | BOOLEAN | Target accepted swap |
| swap_request_id | BIGINT FK | Linked reciprocal request |
| approved_by | BIGINT | Approver ID |
| rejected_by | BIGINT | Rejector ID |

### 3.6 Notification & Communication Tables

#### notifications
In-app notifications

| Column | Type | Description |
|--------|------|-------------|
| id | BIGINT | Primary key |
| employee_id | BIGINT | Target employee |
| type | VARCHAR | Notification type |
| message | TEXT | Notification message |
| data | JSON | Additional data |
| read | BOOLEAN | Read status |

#### device_tokens
Push notification device tokens

| Column | Type | Description |
|--------|------|-------------|
| id | BIGINT | Primary key |
| user_id | BIGINT FK | Employee ID |
| token | VARCHAR | Device token |
| platform | ENUM | 'android', 'ios', 'web' |
| device_name | VARCHAR | Device name |
| device_info | TEXT | Device details |
| last_used_at | TIMESTAMP | Last usage |

#### notification_preferences
Per-employee notification settings

| Column | Type | Description |
|--------|------|-------------|
| id | BIGINT | Primary key |
| employee_id | BIGINT FK | Employee ID |
| check_in_reminder_enabled | BOOLEAN | Check-in reminder |
| check_out_reminder_enabled | BOOLEAN | Check-out reminder |
| leave_status_notification_enabled | BOOLEAN | Leave status updates |
| overtime_notification_enabled | BOOLEAN | Overtime notifications |

#### notification_settings
Global notification settings

| Column | Type | Description |
|--------|------|-------------|
| id | BIGINT | Primary key |
| setting_key | VARCHAR | Unique setting key |
| setting_value | TEXT | Setting value |
| description | VARCHAR | Setting description |

### 3.7 WhatsApp & AI Tables

#### whatsapp_messages
WhatsApp message tracking

| Column | Type | Description |
|--------|------|-------------|
| id | BIGINT | Primary key |
| employee_id | BIGINT FK | Target employee |
| phone | VARCHAR | Recipient phone |
| title | VARCHAR | Message title |
| message | TEXT | Message content |
| type | VARCHAR | Message type (alert, info, etc.) |
| notification_id | BIGINT FK | Related notification |
| message_id | VARCHAR | Gateway message ID |
| status | VARCHAR | 'pending', 'sent', 'delivered', 'read', 'failed' |
| response | TEXT | Gateway response |
| error | TEXT | Error message |
| sent_at | TIMESTAMP | Sent timestamp |
| delivered_at | TIMESTAMP | Delivered timestamp |
| read_at | TIMESTAMP | Read timestamp |

#### wa_gateway_logs
WhatsApp gateway activity logs

| Column | Type | Description |
|--------|------|-------------|
| id | BIGINT | Primary key |
| employee_id | BIGINT FK | Related employee |
| phone | VARCHAR | Phone number |
| direction | VARCHAR | 'incoming', 'outgoing' |
| log_type | VARCHAR | Log category |
| title | VARCHAR | Log title |
| message | TEXT | Log message |
| ai_prompt | TEXT | AI prompt (if applicable) |
| ai_response | TEXT | AI response (if applicable) |
| status | VARCHAR | Processing status |
| message_id | VARCHAR | Gateway message ID |
| gateway_request | TEXT | Request payload |
| gateway_response | TEXT | Response payload |
| error | TEXT | Error details |
| source | VARCHAR | Log source |
| notification_id | BIGINT FK | Related notification |
| wa_message_id | BIGINT FK | Related WhatsApp message |

#### ai_assistant_configs
AI assistant configuration

| Column | Type | Description |
|--------|------|-------------|
| id | BIGINT | Primary key |
| provider | VARCHAR | AI provider ('openai', etc.) |
| api_key | VARCHAR | API key |
| model | VARCHAR | Model name ('gpt-4o', etc.) |
| system_prompt | TEXT | System prompt |
| bot_name | VARCHAR | Bot display name |
| bot_personality | TEXT | Bot personality description |
| context_mode | ENUM | 'none', 'database', 'documents', 'both' |
| max_context_messages | INT | Conversation context length |
| temperature | DECIMAL | Response creativity (0-1) |
| max_tokens | INT | Max response tokens |
| is_active | BOOLEAN | AI assistant enabled |
| is_integrated | BOOLEAN | WhatsApp integration enabled |
| integration_webhook_url | VARCHAR | Webhook URL |
| fallback_enabled | BOOLEAN | Fallback to static responses |
| knowledge_base | TEXT | Custom knowledge base |

#### ai_conversation_histories
WhatsApp AI conversation history

| Column | Type | Description |
|--------|------|-------------|
| id | BIGINT | Primary key |
| employee_id | BIGINT FK | Employee ID |
| phone | VARCHAR | Phone number |
| role | ENUM | 'user', 'assistant' |
| message | TEXT | Message content |
| tokens_used | INT | Token count |
| response_time_ms | INT | Response time |
| success | BOOLEAN | Request success |
| error_message | TEXT | Error details |

#### admin_ai_chat_logs
Admin panel AI chat history

| Column | Type | Description |
|--------|------|-------------|
| id | BIGINT | Primary key |
| user_id | BIGINT FK | Admin user ID |
| role | ENUM | 'user', 'assistant' |
| message | TEXT | Message content |
| ai_response | TEXT | AI response |
| export_type | VARCHAR | Export type (if applicable) |
| export_data | TEXT | Export data (if applicable) |
| tokens_used | INT | Token count |
| response_time_ms | INT | Response time |
| success | BOOLEAN | Request success |
| error_message | TEXT | Error details |

#### lid_phone_mappings
LID (LinkedIn Device) to phone mapping

| Column | Type | Description |
|--------|------|-------------|
| id | BIGINT | Primary key |
| lid | VARCHAR | LID identifier (unique) |
| phone | VARCHAR | Phone number |
| employee_id | BIGINT FK | Related employee |
| is_verified | BOOLEAN | Verification status |
| verified_at | TIMESTAMP | Verification timestamp |

### 3.8 System Tables

#### holidays
Holiday calendar

| Column | Type | Description |
|--------|------|-------------|
| id | BIGINT | Primary key |
| name | VARCHAR | Holiday name |
| date | DATE | Holiday date |
| ket | TEXT | Description |

#### app_logs
Application activity logs

| Column | Type | Description |
|--------|------|-------------|
| id | BIGINT | Primary key |
| lebel | VARCHAR | Log level ('info', 'error', 'warning') |
| message | VARCHAR | Log message |
| context | VARCHAR | Log context |
| ip | VARCHAR | Client IP |
| user_agent | VARCHAR | Browser user agent |
| employee_id | BIGINT FK | Related employee |
| user_id | BIGINT FK | Related admin user |

#### jobs
Queue jobs (Laravel)

| Column | Type | Description |
|--------|------|-------------|
| id | BIGINT | Primary key |
| queue | VARCHAR | Queue name |
| payload | LONGTEXT | Job payload |
| attempts | UNSIGNED TINYINT | Retry attempts |
| reserved_at | INT | Reserved timestamp |
| available_at | INT | Available timestamp |
| created_at | INT | Creation timestamp |

#### cache
Cache storage (Laravel)

| Column | Type | Description |
|--------|------|-------------|
| key | VARCHAR | Cache key (primary) |
| value | MEDIUMTEXT | Cached value |
| expiration | INT | Expiration timestamp |

#### sessions
Session storage (Laravel)

| Column | Type | Description |
|--------|------|-------------|
| id | VARCHAR | Session ID (primary) |
| user_id | BIGINT FK | Related user |
| employee_id | BIGINT FK | Related employee |
| uuid | VARCHAR | Session UUID |
| ip_address | VARCHAR | Client IP |
| user_agent | TEXT | Browser user agent |
| payload | LONGTEXT | Session data |
| last_activity | INT | Last activity timestamp |

#### personal_access_tokens
API tokens (Laravel Sanctum)

| Column | Type | Description |
|--------|------|-------------|
| id | BIGINT | Primary key |
| tokenable_type | VARCHAR | Model type |
| tokenable_id | BIGINT | Model ID |
| name | VARCHAR | Token name |
| token | VARCHAR(64) | Hashed token |
| abilities | TEXT | Token abilities |
| last_used_at | TIMESTAMP | Last usage |
| expires_at | TIMESTAMP | Expiration |

---

## 4. Authentication & Authorization

### 4.1 Authentication Guards

| Guard | Driver | Provider | Usage |
|-------|--------|----------|-------|
| `web` | session | users | Admin panel login |
| `employee` | session | employees | Employee panel login |
| `api` | sanctum | employees | Mobile app API |

### 4.2 Role-Based Access Control

```
+-----------------------------------------------------+
|                    SuperAdmin                        |
|  - Full access to all features                      |
|  - User management (create/delete admins)           |
|  - WA Gateway logs viewer                           |
|  - App logs viewer                                  |
|  - AI Assistant settings                            |
|  - AI Assistant chat interface                      |
+----------------------+------------------------------+
                       |
+----------------------v------------------------------+
|                    Admin                             |
|  - Dashboard (all employees)                        |
|  - Master Data (full CRUD)                          |
|  - Reports (all)                                    |
|  - Export Center                                    |
|  - Notification settings                            |
|  - Import data (Excel)                              |
+----------------------+------------------------------+
                       |
+----------------------v------------------------------+
|                    SDM                               |
|  - Dashboard (limited)                              |
|  - Master Data (view + limited edit)                |
|  - Reports (view only)                              |
+----------------------+------------------------------+
                       |
+----------------------v------------------------------+
|                    Kadiv                             |
|  - Dashboard (own division only)                    |
|  - Approval for division requests                   |
|  - View division reports                            |
+----------------------+------------------------------+
                       |
+----------------------v------------------------------+
|                    Employee                          |
|  - Personal dashboard                               |
|  - Attendance (check-in/out)                        |
|  - Attendance history                               |
|  - Submit requests (leave, overtime, shift swap)    |
|  - View schedule                                    |
|  - Profile management                               |
+-----------------------------------------------------+
```

### 4.3 Middleware Chain

| Middleware | Purpose | Applied To |
|------------|---------|------------|
| `LoginAttemptMiddleware` | Rate limiting on login | Guest routes |
| `CheckAppVersion` | Validate mobile app version | Guest + API routes |
| `RoleBasedSession` | Role-based session isolation | Authenticated routes |
| `CheckSuperAdmin` | SuperAdmin authorization | Admin settings routes |
| `TrustProxies` | Proxy header trust | All routes |
| `CheckDeviceType` | Device type detection | Specific routes |

### 4.4 Session Security

- Session stored in database
- Role-based session separation (user_id vs employee_id)
- UUID tracking for device identification
- Remember token for persistent login
- Login attempt tracking with rate limiting

---

## 5. API Reference

### 5.1 Authentication API

#### POST /api/login
Employee login for mobile app access.

**Request Body:**
```json
{
  "phone": "081234567890",
  "password": "password123"
}
```

**Response (200):**
```json
{
  "token": "1|abc123...",
  "employee": {
    "id": 1,
    "badge": "EMP001",
    "full_name": "John Doe",
    "phone": "081234567890",
    "email": "john@example.com",
    "devisi_id": 1,
    "position_id": 1
  }
}
```

#### POST /api/logout
Logout and revoke token.

**Headers:** `Authorization: Bearer {token}`

### 5.2 Notifications API

#### GET /api/notifications
Get employee notifications.

**Headers:** `Authorization: Bearer {token}`

**Query Parameters:**
- `page` (optional): Page number
- `per_page` (optional): Items per page

**Response (200):**
```json
{
  "data": [
    {
      "id": 1,
      "type": "attendance",
      "message": "Check-in successful",
      "read": false,
      "created_at": "2026-05-17T08:00:00.000000Z"
    }
  ]
}
```

#### POST /api/notifications/{id}/read
Mark notification as read.

#### POST /api/notifications/{id}/respond
Respond to notification action.

### 5.3 Device Token API

#### POST /api/device-tokens
Register device for push notifications.

**Headers:** `Authorization: Bearer {token}`

**Request Body:**
```json
{
  "token": "device_push_token_here",
  "platform": "android",
  "device_name": "Samsung Galaxy S21",
  "device_info": "{\"model\":\"SM-G991B\",\"os\":\"Android 13\"}"
}
```

#### DELETE /api/device-tokens/{token}
Remove device token.

### 5.4 WhatsApp Webhook

#### POST /api/whatsapp/webhook
Receive WhatsApp message status callbacks.

**Request Body:**
```json
{
  "message_id": "msg_123",
  "status": "delivered",
  "phone": "081234567890",
  "timestamp": "2026-05-17T08:00:00Z"
}
```

### 5.5 AI Assistant API

#### GET /api/ai-assistant/context
Get AI assistant context for a phone number.

**Query Parameters:**
- `phone`: Phone number to look up

#### POST /api/ai-assistant/chat
Send chat message to AI assistant.

**Request Body:**
```json
{
  "phone": "081234567890",
  "message": "Apa jadwal saya hari ini?",
  "lid": "lid_abc123"
}
```

### 5.6 Export API

#### POST /api/export/admin
Admin export endpoint.

**Request Body:**
```json
{
  "type": "attendance",
  "start_date": "2026-05-01",
  "end_date": "2026-05-17",
  "format": "excel"
}
```

#### POST /api/export/employee
Employee export endpoint (requires auth:employee).

### 5.7 Onboarding API V2

#### GET /api/v2/onboarding
Get mobile app onboarding structure.

**Response (200):**
```json
{
  "categories": [
    {
      "id": "dashboard",
      "title": "Dashboard",
      "slides": [...]
    }
  ]
}
```

#### GET /api/v2/onboarding/{category}
Get specific onboarding category.

---

## 6. User Guide

### 6.1 Admin Panel Guide

#### Dashboard
The admin dashboard provides:
- Real-time attendance statistics
- Today's attendance summary (present, late, absent)
- Quick actions for common tasks
- Attendance chart (daily/monthly)
- Missed checkout alerts

**Key Metrics:**
- Total employees
- Present today
- Late today
- On leave
- Missed checkout

#### Master Data Management

**Employee Management:**
1. Navigate to Master Data > Employee
2. Click "Add New Employee"
3. Fill required fields:
   - Badge number (unique)
   - Full name
   - Phone number
   - Email
   - Division (Devisi)
   - Position
   - Attendance location
   - Schedule/Shift assignment
4. Click Save

**Division Management:**
1. Navigate to Master Data > Devisi
2. Add/edit divisions
3. Assign Kadiv (division head)
4. View division members

**Attendance Location:**
1. Navigate to Master Data > Lokasi
2. Add location with GPS coordinates
3. Set radius for validation
4. Link to Unit Kerja locations

**Schedule Management:**
1. Navigate to Master Data > Schedule
2. Create schedule template
3. Add patterns for each day of the week
4. Assign to employees

**Shift Management:**
1. Navigate to Master Data > Shift
2. Create shift template
3. Configure shift patterns (start/end time)
4. Create shift groups for date assignments

#### Reports

**Attendance Report:**
1. Navigate to Report > Presensi
2. Filter by date range, division, employee
3. View summary statistics
4. Export to PDF/Excel

**Details Absensi:**
1. Navigate to Report > Details Absensi
2. Select employee and date
3. View detailed attendance record
4. Mark missed checkout as "mangkir" or correct

**Request Report:**
1. Navigate to Report > Request
2. View all pending/approved/rejected requests
3. Approve or reject requests

#### Settings

**Notification Settings:**
- Configure global notification preferences
- Set reminder times
- Enable/disable notification types

**WA Gateway Logs (SuperAdmin):**
- View all WhatsApp message logs
- Filter by status, date, employee
- View message details and errors

**App Logs (SuperAdmin):**
- View application activity logs
- Filter by level, date, user
- Monitor system health

**AI Assistant Settings (SuperAdmin):**
- Configure AI provider and API key
- Set model, temperature, max tokens
- Configure system prompt and personality
- Enable/disable WhatsApp integration
- Manage knowledge base

**Export Center:**
- Batch export attendance data
- Export employee lists
- Export request reports
- Download previously generated exports

### 6.2 Employee Panel Guide

#### Dashboard
The employee dashboard shows:
- Today's attendance status
- Check-in/check-out buttons
- Current schedule/shift info
- Attendance summary (monthly)
- Quick access to history and requests

#### Attendance (Presensi)
1. Navigate to Presensi
2. System validates:
   - Face detection (camera required)
   - GPS location (within radius)
   - Time window (within shift hours)
3. Click "Check In" or "Check Out"
4. Photo captured automatically
5. Confirmation displayed

**Overtime (Presensi Overtime):**
1. Navigate to Presensi Overtime
2. Same validation as regular attendance
3. Records as overtime attendance

#### History
1. Navigate to History
2. Filter by date range
3. View attendance records:
   - Date
   - Check-in time & location
   - Check-out time & location
   - Status (ontime/late/early)
   - Type (present/requested/overtime)

#### Requests (Perizinan)
1. Navigate to Requested
2. Click "New Request"
3. Select request type:
   - **Sick**: Sick leave with attachment
   - **Leave**: Vacation/cuti
   - **Overtime**: Scheduled overtime
   - **Overtime Not Scheduled**: Unscheduled overtime
   - **Dinas Luar**: Official outside duty
   - **Change Shift**: Shift swap with colleague
4. Fill date range and reason
5. Attach supporting document (photo)
6. Submit and wait for approval

**Shift Swap:**
1. Select "Change Shift" type
2. Select target employee
3. Specify target's shift details
4. Submit request
5. Target employee receives notification
6. Target accepts/declines
7. If accepted, forwarded to admin for approval

#### Schedule
1. Navigate to Schedule
2. View current schedule/shift
3. See daily pattern (start/end time)
4. View shift group assignments

#### Profile
1. Navigate to Profile
2. View personal information
3. Update profile photo
4. View division and position info
5. View attendance statistics

---

## 7. WhatsApp Gateway Integration

### 7.1 Architecture

```
+--------------+     +------------------+     +------------------+
|  Application |---->| WhatsApp Gateway |---->|  WhatsApp API    |
|  (Laravel)   |     |  Service         |     |  (External)      |
+--------------+     +------------------+     +------------------+
       |                                              |
       |              +------------------+            |
       +--------------|  Webhook Handler |<-----------+
                      |  (Status Update) |
                      +------------------+
```

### 7.2 Configuration

**config/whatsapp.php:**
```php
return [
    'gateway_url' => env('WA_GATEWAY_URL'),
    'api_key' => env('WA_GATEWAY_API_KEY'),
    'webhook_secret' => env('WA_WEBHOOK_SECRET'),
];
```

### 7.3 Message Flow

1. **Outgoing Message:**
   - Application creates WhatsAppMessage record
   - WhatsAppGatewayService sends to gateway
   - Gateway returns message_id
   - Status updated to 'sent'

2. **Status Update (Webhook):**
   - Gateway sends POST to /api/whatsapp/webhook
   - Webhook validates secret
   - Updates message status (delivered/read/failed)
   - Creates WA gateway log entry

3. **Incoming Message (AI Chat):**
   - Gateway forwards message to webhook
   - System looks up employee by phone/LID
   - AI Assistant processes message
   - Response sent back via gateway

### 7.4 Message Types

| Type | Description | Trigger |
|------|-------------|---------|
| `alert` | Important notifications | Missed checkout, approval status |
| `info` | Informational messages | Daily reminders |
| `attendance` | Attendance confirmations | Check-in/out success |
| `request` | Request notifications | New request, approval result |
| `ai_response` | AI assistant responses | WhatsApp AI chat |

### 7.5 LID Phone Mapping

LID (LinkedIn Device ID) mapping allows identifying employees by their WhatsApp device:

1. Employee sends message from WhatsApp
2. System extracts LID from message
3. Looks up lid_phone_mappings table
4. If found, identifies employee
5. If not found, prompts for verification

---

## 8. AI Assistant System

### 8.1 Architecture

```
+--------------+     +------------------+     +------------------+
|  User Input  |---->| AI Assistant     |---->| AI Provider      |
|  (WhatsApp/  |     | Service          |     | (OpenAI/etc.)    |
|   Web)       |     |                  |     |                  |
+--------------+     +--------+---------+     +------------------+
                              |
                    +---------v---------+
                    |   Context Builder  |
                    |  - Employee data   |
                    |  - Attendance      |
                    |  - Schedule        |
                    |  - Knowledge base  |
                    +-------------------+
```

### 8.2 Configuration

AI Assistant can be configured via:
- Admin Panel > Settings > AI Assistant (SuperAdmin)
- Direct database edit on ai_assistant_configs table

**Key Settings:**
| Setting | Description | Default |
|---------|-------------|---------|
| provider | AI provider name | 'openai' |
| model | Model identifier | 'gpt-4o' |
| system_prompt | System instructions | Custom |
| bot_name | Display name | 'AI Assistant GSG' |
| context_mode | Context source | 'database' |
| temperature | Response creativity | 0.7 |
| max_tokens | Max response length | 1000 |

### 8.3 Context Modes

| Mode | Description |
|------|-------------|
| `none` | No context, pure AI responses |
| `database` | Includes employee data, attendance, schedule |
| `documents` | Includes knowledge base documents |
| `both` | Database + documents context |

### 8.4 Admin AI Chat

SuperAdmin can use AI Assistant in admin panel:
- Floating chat widget
- Export data via AI commands
- Query system information
- Generate reports

### 8.5 WhatsApp AI Chat

Employees can chat with AI via WhatsApp:
1. Send message to configured WhatsApp number
2. System identifies employee by phone/LID
3. AI processes with context
4. Response sent back via WhatsApp

**Sample Commands:**
- "What is my schedule today?" - Get today's schedule
- "When did I check in?" - Get check-in time
- "How many days was I absent this month?" - Monthly attendance summary
- "What is my request status?" - Request status

---

## 9. Notification System

### 9.1 Notification Channels

| Channel | Description | Configuration |
|---------|-------------|---------------|
| In-App | Notifications within application | Always enabled |
| Push (Web) | Browser push notifications | VAPID keys required |
| Push (Mobile) | Mobile app push notifications | Device tokens |
| WhatsApp | WhatsApp messages | Gateway configured |

### 9.2 Scheduled Notifications

**Attendance Reminders:**
- Runs every minute via `notifications:send-attendance`
- Checks employee schedules
- Sends reminders before check-in/check-out time
- Respects notification preferences

**Notification Flow:**
```
1. Scheduled task runs every minute
2. Fetches employees with upcoming check-in/out
3. Checks notification preferences
4. Sends via enabled channels:
   - Push notification (web/mobile)
   - WhatsApp (if enabled)
5. Logs notification activity
```

### 9.3 Notification Types

| Type | Trigger | Channels |
|------|---------|----------|
| Check-in Reminder | Before shift start | Push, WhatsApp |
| Check-out Reminder | Before shift end | Push, WhatsApp |
| Attendance Success | After check-in/out | In-App, WhatsApp |
| Request Status | On approval/rejection | In-App, Push, WhatsApp |
| Missed Checkout | End of day, no checkout | In-App, WhatsApp |
| Shift Swap Request | When swap requested | In-App, Push, WhatsApp |

### 9.4 Push Notification Setup

**Web Push (VAPID):**
```env
VAPID_PUBLIC_KEY=your_public_key
VAPID_PRIVATE_KEY=your_private_key
VAPID_SUBJECT=mailto:admin@example.com
```

**Generate VAPID Keys:**
```bash
php artisan vendor:publish --provider="NotificationChannels\WebPush\WebPushServiceProvider"
```

---

## 10. Attendance System

### 10.1 Attendance Types

| Type | Description |
|------|-------------|
| `present` | Regular attendance |
| `requested` | Attendance during approved request |
| `changeSched` | Attendance with schedule change |
| `overtime` | Scheduled overtime attendance |
| `overtimeNotSched` | Unscheduled overtime attendance |

### 10.2 Validation Rules

**Check-in Validation:**
1. Face detection via face-api.js
2. GPS location within radius of assigned attend_location
3. GPS accuracy within threshold
4. Time within shift start window
5. Not a holiday (unless shift ignores holidays)
6. Employee status is active (0)

**Check-out Validation:**
1. Must have check-in record for the day
2. Face detection
3. GPS location validation
4. Time validation (early departure tracking)

### 10.3 Late/Early Calculation

**Late Calculation:**
```
late_time = attd_in_time - shift_pattern.start_time
if late_time > 0: status_in = 'late'
else: status_in = 'ontime'
```

**Early Departure Calculation:**
```
early_time = shift_pattern.end_time - attd_out_time
if early_time > 0: status_out = 'early'
else: status_out = 'ontime'
```

### 10.4 Missed Checkout Handling

When employee forgets to check out:
1. Admin sees missed checkout in dashboard
2. Options:
   - **Mark as Mangkir**: Mark as absent for the day
   - **Koreksi**: Manually set check-out time
3. Correction logged in app_logs

### 10.5 Face Recognition

**Technology:** face-api.js (TensorFlow.js)
**Models:** tiny_face_detector
**Process:**
1. Camera captures video stream
2. face-api.js detects face in real-time
3. When face detected, captures photo
4. Photo uploaded with attendance record
5. Server-side validation (optional)

---

## 11. Shift & Schedule Management

### 11.1 Schedule System

**Schedule Template:**
- Named collection of daily patterns
- Example: "Regular Office Hours"

**Schedule Pattern:**
- Daily time configuration
- Example: Monday-Friday 08:00-17:00

**Assignment:**
- Employee assigned to one schedule
- System determines daily pattern from schedule

### 11.2 Shift System

**Shift Template:**
- Named shift definition
- Example: "Morning Shift", "Night Shift"

**Shift Pattern:**
- Time configuration for shift
- Properties:
  - start_time, end_time
  - next_day (overnight shift)
  - can_change (allow swap requests)
  - ignore_holiday (active on holidays)

**Shift Group:**
- Date-specific shift assignment
- Allows rotating shifts
- Example: Employee A on Morning Shift on May 17, Night Shift on May 18

### 11.3 Shift Rotation

```
Shift Pattern A (08:00-16:00)
Shift Pattern B (16:00-00:00)
Shift Pattern C (00:00-08:00)

Shift Group Assignment:
Date: 2026-05-17 -> Shift Pattern A
Date: 2026-05-18 -> Shift Pattern B
Date: 2026-05-19 -> Shift Pattern C
```

### 11.4 Holiday Handling

- Holidays fetched automatically via API (December 28 each year)
- Stored in holidays table
- Shift patterns can ignore holidays (ignore_holiday = true)
- Regular schedules skip holidays

---

## 12. Request & Approval Workflow

### 12.1 Request Types

#### Schedule Request Types
| Type | Description | Approval Required |
|------|-------------|-------------------|
| sick | Sick leave | Yes |
| leave | Vacation/cuti | Yes |
| overtime | Scheduled overtime | Yes |
| overtimeNotSched | Unscheduled overtime | Yes |
| dinas_luar | Official outside duty | Yes |

#### Shift Request Types
| Type | Description | Approval Required |
|------|-------------|-------------------|
| present | Present (outside location) | Yes |
| leave | Vacation/cuti | Yes |
| sick | Sick leave | Yes |
| change_shift | Shift swap | Yes (dual approval) |
| overtime | Scheduled overtime | Yes |
| overtimeNotSched | Unscheduled overtime | Yes |
| dinas_luar | Official outside duty | Yes |

### 12.2 Approval Hierarchy

```
Employee submits request
        |
        v
+------------------+
|  Division Head   |  (if Kadiv assigned)
|  (Kadiv)         |
+--------+---------+
         |
         v
+------------------+
|  Admin/SDM       |  (final approval)
+--------+---------+
         |
         v
+------------------+
|  Notification    |
|  to Employee     |
+------------------+
```

### 12.3 Shift Swap Process

```
Employee A requests swap with Employee B
        |
        v
+------------------------------+
|  Request created (pending)   |
|  - target_employee_id: B     |
|  - swap_request_id: created  |
+--------------+---------------+
               |
               v
+------------------------------+
|  Employee B notified         |
|  - WhatsApp + in-app         |
|  - Can accept or decline     |
+--------------+---------------+
               |
        +------+------+
        v             v
+--------------+ +--------------+
|  Accepted    | |  Declined    |
|  (forwarded  | |  (cancelled) |
|   to admin)  | |              |
+------+-------+ +--------------+
       |
       v
+------------------------------+
|  Admin approves              |
|  - Both shift groups updated |
|  - Both employees notified   |
+------------------------------+
```

---

## 13. Export & Reporting

### 13.1 Export Types

| Export | Description | Format |
|--------|-------------|--------|
| Attendance | Individual attendance records | PDF, Excel |
| All Attendance Summary | Summary of all employees | PDF, Excel |
| Employee List | Employee master data | PDF, Excel |
| Late Summary | Late attendance summary | PDF |
| Requests | Request reports | PDF |

### 13.2 Export Center

Admin can batch export data:
1. Navigate to Settings > Export Center
2. Select export type
3. Configure filters (date range, division, etc.)
4. Generate export
5. Download from export list

### 13.3 PDF Templates

Located in `resources/views/exports/`:
- `attendance-pdf.blade.php` - Individual attendance
- `all-attendance-summary-pdf.blade.php` - Summary
- `employee-attendance-pdf.blade.php` - Employee-specific
- `employees-pdf.blade.php` - Employee list
- `late-summary-pdf.blade.php` - Late summary
- `requests-pdf.blade.php` - Request report

---

## 14. Deployment Guide

### 14.1 Server Requirements

| Component | Minimum | Recommended |
|-----------|---------|-------------|
| PHP | 8.3 | 8.3+ |
| MySQL | 8.0 | 8.0+ |
| RAM | 2GB | 4GB+ |
| Storage | 5GB | 10GB+ |
| SSL | Required | Let's Encrypt |

### 14.2 Deployment Steps

1. **Prepare Server:**
```bash
# Install PHP 8.3+
sudo apt update
sudo apt install php8.3 php8.3-fpm php8.3-mysql php8.3-mbstring php8.3-xml php8.3-curl php8.3-zip php8.3-gd

# Install Composer
curl -sS https://getcomposer.org/installer | php
sudo mv composer.phar /usr/local/bin/composer

# Install Node.js
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install nodejs
```

2. **Clone & Setup:**
```bash
cd /var/www
git clone https://github.com/Ismail-IndoTech/IndoTech-Absensi_V2.git
cd IndoTech-Absensi_V2

composer install --optimize-autoloader --no-dev
npm install
npm run build

cp .env.example .env
php artisan key:generate
```

3. **Configure Environment:**
```env
APP_NAME="IndoTech Presensi Online"
APP_ENV=production
APP_DEBUG=false
APP_URL=https://absensi.yourdomain.com

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=indotech_absensi
DB_USERNAME=absensi_user
DB_PASSWORD=GANTI_DENGAN_PASSWORD_KUAT

# Queue
QUEUE_CONNECTION=database

# WhatsApp Gateway
WA_GATEWAY_URL=https://your-wa-gateway.com
WA_GATEWAY_API_KEY=your_api_key
WA_WEBHOOK_SECRET=your_secret

# AI Assistant
AI_PROVIDER=openai
AI_API_KEY=your_openai_key
```

4. **Database Setup:**
```bash
php artisan migrate --force
php artisan db:seed --force
php artisan storage:link
```

5. **Web Server (Nginx):**
```nginx
server {
    listen 80;
    server_name absensi.yourdomain.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name absensi.yourdomain.com;

    ssl_certificate /etc/letsencrypt/live/absensi.yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/absensi.yourdomain.com/privkey.pem;

    root /var/www/IndoTech-Absensi_V2/public;
    index index.php;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php8.3-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.(?!well-known) {
        deny all;
    }
}
```

6. **Queue Worker:**
```bash
# Create systemd service
sudo nano /etc/systemd/system/absensi-queue.service
```

```ini
[Unit]
Description=IndoTech Absensi Queue Worker
After=network.target

[Service]
Type=simple
User=www-data
WorkingDirectory=/var/www/IndoTech-Absensi_V2
ExecStart=/usr/bin/php artisan queue:work --tries=3 --timeout=60
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl enable absensi-queue
sudo systemctl start absensi-queue
```

7. **Scheduled Tasks (Cron):**
```bash
crontab -e
```

Add:
```cron
* * * * * cd /var/www/IndoTech-Absensi_V2 && php artisan schedule:run >> /dev/null 2>&1
```

### 14.3 Post-Deployment

1. Run optimization:
```bash
php artisan optimize
php artisan view:cache
php artisan route:cache
php artisan config:cache
```

2. Create first admin user via seeder or manually

3. Configure WhatsApp gateway credentials

4. Test attendance flow

5. Set up backup strategy

---

## 15. Troubleshooting

### 15.1 Common Issues

**Issue: Login fails**
- Check database connection
- Verify employee status = 0 (active)
- Check login attempt middleware (rate limiting)

**Issue: Face detection not working**
- Ensure HTTPS (camera requires secure context)
- Check browser permissions
- Verify face-api.js models loaded in /models/

**Issue: GPS location validation fails**
- Check browser location permissions
- Verify attend_location coordinates
- Check radius configuration
- Test with /debug-ip endpoint

**Issue: WhatsApp messages not sent**
- Check WA_GATEWAY_URL configuration
- Verify API key
- Check wa_gateway_logs for errors
- Test webhook endpoint

**Issue: Queue jobs not processing**
- Check queue worker is running: `systemctl status absensi-queue`
- Check failed_jobs table
- Restart queue worker: `systemctl restart absensi-queue`

**Issue: Scheduled tasks not running**
- Verify cron is configured
- Check cron logs: `grep CRON /var/log/syslog`
- Test manually: `php artisan schedule:run`

### 15.2 Debug Commands

```bash
# Check application logs
tail -f storage/logs/laravel.log

# Check queue status
php artisan queue:monitor database

# View failed jobs
php artisan queue:failed

# Retry failed jobs
php artisan queue:retry all

# Clear cache
php artisan cache:clear
php artisan config:clear
php artisan route:clear
php artisan view:clear

# Database check
php artisan db:show
php artisan db:table employees

# Route list
php artisan route:list

# Check scheduled tasks
php artisan schedule:list
```

### 15.3 Performance Optimization

1. **Database Indexes:** Already configured via migrations
2. **Query Caching:** DashboardCacheService for dashboard queries
3. **Asset Optimization:** Vite build with code splitting
4. **Queue Processing:** Database queue with retry logic
5. **Session Optimization:** Database sessions with cleanup

### 15.4 Backup Strategy

**Database Backup:**
```bash
mysqldump -u username -p indotech_absensi > backup_$(date +%Y%m%d).sql
```

**File Backup:**
```bash
tar -czf backup_$(date +%Y%m%d).tar.gz \
  --exclude=vendor \
  --exclude=node_modules \
  --exclude=storage/logs \
  /var/www/IndoTech-Absensi_V2
```

**Automated Backup (Cron):**
```cron
0 2 * * * mysqldump -u user -p'password' indotech_absensi | gzip > /backups/db_$(date +\%Y\%m\%d).sql.gz
```

---

## Appendix A: Environment Variables Reference

| Variable | Description | Required | Default |
|----------|-------------|----------|---------|
| APP_NAME | Application name | Yes | "IndoTech Presensi Online" |
| APP_ENV | Environment | Yes | production |
| APP_DEBUG | Debug mode | Yes | false |
| APP_URL | Application URL | Yes | - |
| DB_CONNECTION | Database driver | Yes | mysql |
| DB_HOST | Database host | Yes | 127.0.0.1 |
| DB_DATABASE | Database name | Yes | - |
| DB_USERNAME | Database user | Yes | - |
| DB_PASSWORD | Database password | Yes | - |
| WA_GATEWAY_URL | WhatsApp gateway URL | No | - |
| WA_GATEWAY_API_KEY | WhatsApp API key | No | - |
| WA_WEBHOOK_SECRET | Webhook secret | No | - |
| AI_PROVIDER | AI provider name | No | openai |
| AI_API_KEY | AI API key | No | - |
| VAPID_PUBLIC_KEY | Web push public key | No | - |
| VAPID_PRIVATE_KEY | Web push private key | No | - |
| VAPID_SUBJECT | Web push subject | No | - |

---

## Appendix B: Artisan Commands

| Command | Description |
|---------|-------------|
| `notifications:send-attendance` | Send attendance reminders |
| `logs:delete-old` | Delete old application logs |
| `holidays:fetch` | Fetch holidays from API |
| `optimize` | Cache config, routes, views |
| `storage:link` | Create storage symlink |
| `migrate` | Run database migrations |
| `db:seed` | Seed database |
| `queue:work` | Start queue worker |
| `cache:clear` | Clear application cache |

---

*End of TechDock Documentation*

For questions or updates, contact the development team or create an issue in the repository.
