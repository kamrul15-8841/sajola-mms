# সজলা - Mess Management System

**Sojola - Mess Management System** (Laravel 10)

A mess (shared kitchen/canteen) management system for a factory/office. Members log their daily meals (lunch/dinner), managers record daily expenses and menus, and the system tracks meal rates, deposits, and member balances automatically. The UI is Bangla (বাংলা) with employee details auto-fetched from an internal HR API.

This is a **Laravel 10 rewrite** of an original custom PHP (procedural micro-framework) application. The original lives at `E:\laragon\www\php74\sojola-meal-management-system` and is kept untouched as a reference.

---

## Quick Start

Requirements:

- PHP 8.1+
- MySQL (or MariaDB)
- Composer

```bash
# 1. Install dependencies
composer install

# 2. Configure environment
copy .env.example .env          # Windows
php artisan key:generate

# 3. Set database connection in .env
#    DB_DATABASE=sajola_mms
#    DB_USERNAME=root
#    DB_PASSWORD=yourpassword
#    APP_TIMEZONE=Asia/Dhaka   (already set)
#    MESS_EMPLOYEE_MEAL_RATE=70  (optional, default 70)

# 4. Create database + tables + seed demo data
php artisan migrate:fresh --seed

# 5. Run the development server
php artisan serve --port=8075
```

Open `http://127.0.0.1:8075` in your browser.

---

## Demo Accounts (created by the seeder)

| Role         | Card No / ID | Password |
|--------------|--------------|----------|
| Superadmin   | `ADMIN001`   | `admin123` |
| Manager      | `MGR-001`    | `123456`  |
| Member       | `EMP-001`    | `123456`  |
| Member       | `EMP-002`    | `123456`  |
| Member       | `EMP-003`    | `123456`  |
| Member       | `EMP-004`    | `123456`  |

Superadmin = full access (settings, activity-log restore, server setup, all reports).
Manager = all day-to-day admin pages (dashboard, meals, expenses, menus, members, users, meal-rate report).
Member = own dashboard only (log own meals, add deposit, view own balance and menus).

---

## What the System Does

**Member portal (`/member/...`)**
- Log own lunch/dinner meals day by day (only today is editable).
- Add a deposit / advance payment.
- See own total meals, total deposit, meal-rate, balance, today's meal list and the cooking menu.
- Change password.

**Admin portal (`/admin/...`)**
- Dashboard: today's summary — total meals (lunch/dinner), today's expense, running meal rate, deposits, balances, quick tabs to enter meals, expenses and menus.
- Meals: view meals filtered by Today / Month / Year / custom date range, add meals for members, bulk-mark whole days, edit/delete entries.
- Expenses: same date filtering, add/edit/delete daily expense items (৳ amounts + note).
- Menus: same date filtering, add/edit/delete the cooking menu per day (lunch/dinner text).
- Members: card search (HR API autofill), add/edit/delete members and managers, add deposit on behalf of a member.
- User list: all users with meals, deposit and balance summary.
- Meal Rate Report: per-member and per-manager meal-rate report including employee/company rates.
- CSV import: download an Excel/CSV template and import many users at once.
- Settings (superadmin): toggle each feature on/off and restrict features to superadmin only.
- Setup (superadmin): server/DB settings + employee meal rate (saved to `.env`).
- Activity Log (superadmin): view all logged actions and restore deleted records.

---

## Key Concepts

- **Meal rate** for a period = `total expense ÷ total meals` in that period.
- **Employee rate** is a fixed company-subsidised rate (default `70` Taka, changeable in Setup / `.env` `MESS_EMPLOYEE_MEAL_RATE`).
- **Company subsidy** = `(meal_rate − employee_rate) × meals` — the company absorbs the rest.
- **Member balance** = `total deposit − (meals × employee_rate)`. Paid members show a positive balance; members who ate more than they paid show negative (due).
- Rate depends on the **role**: `employee` members use the employee rate, `manager` (and `superadmin`) use the fuller company rate.
  - Wait – see `MessService::memberCounts()`, `MessService::memberBalance()` and `MessService::dailyMealRates()` for the exact rules; the above is the general contract.

**Feature flags** are read from the `settings` table at runtime (`Feature::isActive('meal_entry')` etc.). A feature that is off hides its sidebar link and its tabs/forms. A `<flag>_sa` setting of `1` restricts that feature to superadmin only.

---

## Tech Stack

- **PHP 8.1** / **Laravel 10** (`laravel/framework ^10.0`)
- **MySQL** database (`sajola_mms`)
- **Blade** templates (Bangla UI, CSS/JS copied from the original app — no frontend build step)
- `laravel/sanctum` (installed, not required for normal usage)
- Session driver `file`, timezone `Asia/Dhaka`
- No queues, no cache drivers, no jobs, no notifications — the app is intentionally simple.

---

## Project Layout (the parts that matter)

```
sajola-mms/
├── app/
│   ├── Http/
│   │   ├── Controllers/   AdminController, AuthController, MemberController, Controller (base)
│   │   └── Middleware/    EnsureRole, EnsureUserIsAuthenticated, ShareViewData, …
│   ├── Models/            User, DailyMeal, Deposit, Expense, Menu, Setting, ActivityLog
│   ├── Services/          MessService (business logic), Feature (flags), EmployeeApi (HR API)
│   └── Support/Config.php App name/version/timezone/employee rate helpers
├── config/mess.php        mess.* config (employee_meal_rate)
├── database/migrations/   7 domain tables + Laravel's personal_access_tokens
├── database/seeders/      DatabaseSeeder (users, deposits, expenses, meals, menus, settings)
├── public/
│   ├── css/style.css      The original stylesheet
│   └── js/app.js          The original JS (modals, toast, card search, …)
├── resources/views/       Blade views (layouts/app, layouts/main, auth, admin, member)
└── routes/web.php         ALL routes
```

---

## Documentation

Detailed guides live in [docs/](docs/):

| Doc | Covers |
|-----|--------|
| [01-conversion-overview](docs/01-conversion-overview.md) | How and why this was ported from the original custom PHP app |
| [02-technical-architecture](docs/02-technical-architecture.md) | Layers, request flow, where each responsibility lives |
| [03-database-schema](docs/03-database-schema.md) | Every table, column, type, key and how tables relate |
| [04-models](docs/04-models.md) | Each Eloquent model, fillables, casts, methods, scopes |
| [05-business-logic-services](docs/05-business-logic-services.md) | MessService calculations, Feature, EmployeeApi, Config |
| [06-routes-and-controllers](docs/06-routes-and-controllers.md) | Every route, controller action, and old→new URL map |
| [07-authentication-and-roles](docs/07-authentication-and-roles.md) | Login, session, roles, redirect rules, CSRF |
| [08-middleware-and-request-lifecycle](docs/08-middleware-and-request-lifecycle.md) | Middleware order and shared view data |
| [09-views-and-frontend](docs/09-views-and-frontend.md) | Blade structure, layouts, JS/CSS wiring, flash messages |
| [10-settings-and-feature-flags](docs/10-settings-and-feature-flags.md) | The settings table, all flags, superadmin restriction |
| [11-activity-log-and-restore](docs/11-activity-log-and-restore.md) | Who did what, and how restore works |
| [12-import-and-employee-api](docs/12-import-and-employee-api.md) | CSV user import + HR employee API |
| [13-seeding-test-data](docs/13-seeding-test-data.md) | What the seeder creates and why |
| [14-testing-and-verification](docs/14-testing-and-verification.md) | Manual test checklist, artisan commands, log locations |
| [15-maintaining-and-adding-features](docs/15-maintaining-and-adding-features.md) | Step-by-step recipe for new features + conventions |
| [16-troubleshooting](docs/16-troubleshooting.md) | Common issues and fixes (incl. known Laravel gotchas) |

---

## Useful Artisan Commands

```bash
php artisan migrate:fresh --seed   # reset DB + reseed demo data
php artisan serve --port=8075      # dev server
php artisan storage:link           # if you ever serve uploaded files
php artisan tinker                 # REPL to inspect/inject data
php -l app/Http/Controllers/AdminController.php  # lint a single file
```

## Logs

- Application errors: `storage/logs/laravel.log`
- The legacy app also wrote plain-PHP logs — see the original project if you need `logs/` from there.

## Environment Variables

| Variable | Default | Purpose |
|----------|---------|---------|
| `APP_TIMEZONE` | `Asia/Dhaka` | All dates are shown in Dhaka time |
| `MESS_EMPLOYEE_MEAL_RATE` | `70` | Employee (subsidized) meal rate in ৳ |
| `SESSION_DRIVER` | `file` | Session store (file is fine for this app) |
| `DB_*` | — | Database connection |

> `MESS_EMPLOYEE_MEAL_RATE` is configurable at runtime from **Setup → Employee Meal Rate** (superadmin). See [docs/05](docs/05-business-logic-services.md) and [docs/10](docs/10-settings-and-feature-flags.md).

---

## License

MIT (Laravel's default license). This is an internal company tool; no external distribution planned.
