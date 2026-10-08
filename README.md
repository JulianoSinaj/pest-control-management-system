# Pest Control Management System

Management software for a pest control and rodent control company.
Required stack: plain PHP + MySQL, an HTML/CSS/JavaScript + jQuery (AJAX) frontend, mPDF for PDF generation, and `html5-qrcode` for QR scanning.

## Requirements
- XAMPP (Apache + MySQL + PHP 8). The project is located at `C:\XAMPP\htdocs\insetto`.
- Enable the PHP **gd** extension (required by mPDF). In `C:\XAMPP\php\php.ini`, make sure the `extension=gd` line is not commented out.
- Install mPDF in `/vendor` using Composer.

## Setup
```bash
# 1. Create the database and base tables
C:\XAMPP\mysql\bin\mysql.exe -u root < schema.sql

# 1b. Add inventory, HACCP, SDI invoicing, and the supervisor role
C:\XAMPP\mysql\bin\mysql.exe -u root < schema_v2.sql

# 2. Install PHP dependencies (mPDF)
composer install

# 3. Create initial users and configuration data
C:\XAMPP\php\php.exe seed.php
```
Then open **http://localhost/insetto/**. It redirects to the login page.

## Default Credentials (created by `seed.php`)
| Role       | Email                   | Password  |
|------------|-------------------------|-----------|
| admin      | admin@deltasoftware.it  | admin123  |
| supervisor | super@deltasoftware.it  | super123  |
| technician | tecnico@deltasoftware.it| tecnico123|

The four roles are defined in `src/rbac.php`:
- **admin**: full access, including user management.
- **supervisor**: full operational access (clients, locations, interventions, inventory, HACCP, invoicing, and dashboard), except user management.
- **technician**: field app access and inventory usage logging for their own interventions.
- **client**: read-only access to their own data.

> Change the default passwords after the first login. Create **client** accounts
> in the management system under *Users* and associate them with a client.

## Project Structure
```
/public            web entry point (login, admin system, client portal)
/public/api        AJAX endpoints (JSON)
/public/assets     CSS and JavaScript (core and views)
/frontend          mobile field app for technicians, including QR scanning
/src               PHP logic (database, authentication, helpers, thresholds, reports)
/vendor            Composer dependencies, including mPDF
/storage           photo uploads and PDF reports (not web-accessible)
config.php         database credentials and company settings (not version controlled)
schema.sql         database schema
seed.php           initial data
```

## Roles and Routing (Single Login)
The login page redirects users based on their role:
- **admin** → `/public/index.php` (full management system)
- **technician** → `/frontend/` (field app)
- **client** → `/public/portale.php` (read-only portal)

## Implemented Modules
1. Database schema and PDO connection
2. Authentication, roles, and CSRF protection (PHP sessions)
3. Client records (cards + AJAX)
4. Locations → areas → numbered stations with QR codes
5. Field QR scanning (`html5-qrcode`)
6. Intervention scheduling and technician app (readings, maintenance, evidence, validation)
7. Device type configuration and automatic threshold calculation
8. PDF reports (mPDF) with a custom layout and email delivery
9. Read-only client portal (intervention history, reports, and stations)

## V2 Extensions (Beyond the MVP)
- **Inventory/Warehouse**: products, batches, expiration dates, reorder thresholds, alerts, and product usage per intervention (FEFO stock deduction). API: `magazzino.php`; view: *Inventory*.
- **HACCP/IPM**: automatic log of threshold exceedances, populated when an intervention is validated. API: `haccp.php`; view: *HACCP*.
- **SDI Invoicing**: quotes and invoices with line items, taxable amount/VAT/total calculations, and **FatturaPA XML** generation (partial field coverage; *not signed or transmitted*). API: `documenti.php`; view: *Invoices*.
- **KPI Dashboard**: interventions by technician/area, stations over threshold, and infestation trends. API: `dashboard.php`; view: *Dashboard*.
- **Four-role RBAC** centralized in `src/rbac.php` (`require_can('area.action')`).

## Security
- PDO prepared statements for database queries
- Passwords handled with `password_hash()` / `password_verify()`
- Role checks on every endpoint; clients can only access their own data
- CSRF tokens (`X-CSRF-Token`) on all data-changing requests
- `.htaccess` blocks direct access to `config.php`, `schema.sql`, `/src`, `/storage`, and `/vendor`

## Notes
- Report emails use `mail()`. In a local XAMPP setup without SMTP, email delivery will not work, but the PDF is still generated and saved. Configure SMTP for production.
- The reference files `docs/report_esempio.pdf` and the UI images were not provided, so the report layout follows the structure described in the specification.
