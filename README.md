# Career Recommendation System (PHP + MySQL)

Minimal MVP that matches the uploaded FYP spec: signup/login, assessment, rule-based recommendations, feedback, and a basic admin career catalog.

## Requirements
- PHP 8.1+ with PDO MySQL
- MySQL 8+
- Apache or PHP built-in server
- VS Code (or any editor)

## Setup
1) Create database & tables:
   - Import `data/schema.sql` into MySQL

2) Configure DB:
   - Edit `app/config.php` if your DB host/user/pass differ
   - Or set env vars: DB_HOST, DB_NAME, DB_USER, DB_PASS

3) Run the app locally:
   ```bash
   cd public
   php -S 127.0.0.1:8000
   ```
   Open http://127.0.0.1:8000 in your browser.

4) Create an admin (optional):
   ```sql
   UPDATE users SET role='admin' WHERE email='YOUR_EMAIL';
   ```

## Files
- `public/` routes: `login.php`, `register.php`, `dashboard.php`, `assessment.php`, `recommendations.php`, `profile.php`, `logout.php`, `admin_careers.php`
- `app/config.php` DB + session + helpers
- `app/partials/` header/footer
- `data/schema.sql` DB schema + seed careers
- `public/assets/styles.css` basic styling

## Notes
- Recommendation scoring is rule-based and deterministic. Replace with ML later if needed.
- Sessions use PHP defaults; for production, set secure cookie params.
- CSRF token is included on forms.


## New features
- Dynamic question bank with multi‑step assessment and progress bar (`assessment_quiz.php`, `questions`, `assessment_answers` tables).
- Weighted scoring → feeds recommendations.
- Admin dashboard (`admin_dashboard.php`).
- Admin question management (`admin_questions.php`).
- CSV exports: `export_feedback.php`, `export_recommendations.php`.
- Extra schema: `data/schema_add.sql` (run this after `schema.sql`).

## Upgrade steps (if you already set up earlier MVP)
1) Import `data/schema_add.sql`.
2) Use **Assessment** via `/assessment_quiz.php` (nav updated).
3) Optional: Visit `/admin_dashboard.php`, `/admin_questions.php`.
4) CSV exports: `/export_feedback.php`, `/export_recommendations.php` (admin only).

## PDF export
- Library: bundled lightweight `vendor/fpdf/fpdf.php` (no Composer needed).
- User report: `/export_pdf.php` (logged-in user only)
- Admin export by user: `/export_user_pdf.php?user_id=ID`

If you deploy behind Apache/Nginx, ensure PHP can send headers (no prior output).

## AI microservice
- Location: `ml/app.py` (Flask). Endpoints:
  - `POST /predict` with `{"X":[[interest_overlap, skill_overlap, personality_match], ...]}` → `{scores:[0..1]}`
  - `POST /fit` with `{"X": [...], "y": [0/1]}` retrains and saves `model.joblib`
- Start:
  ```bash
  python3 -m venv .venv && source .venv/bin/activate
  pip install -r ml/requirements.txt
  python ml/app.py
  ```

## Integration
- Recommendations call the AI first; if offline, rules are used.
- Admin → Retrain builds training data from feedback.

## Security additions
- Security headers (CSP, XFO, etc.), simple per-route session rate limit.
- Prepared statements + CSRF already in place.

## System metrics
- `/admin_metrics.php` shows basic load & memory (Linux).

## Extra indexes
- Run `data/schema_add2.sql` for better performance.

## Production features (v5)
- Save/Resume assessment (DB-backed) + autosave JS.
- Email flows: verification & password reset (token tables, `mail()` fallback).
- Audit logs for key actions.
- Prometheus metrics at `/metrics.php`.
- Labor stats stub shown on recommendations.
- Docker Compose: Nginx + PHP-FPM + MySQL + ML + Prometheus + Grafana.
- Queue worker for scheduled jobs (`cron/worker.php`), jobs table.
- PHPUnit scaffold.
- Privacy/ToS/Retention pages.
- Extra schema: `data/schema_add3.sql`.

### Docker quickstart
```
docker compose up -d
# import SQL files into db container
docker exec -i $(docker ps -qf name=db) mysql -uroot -pexample < data/schema.sql
docker exec -i $(docker ps -qf name=db) mysql -uroot -pexample < data/schema_add.sql
docker exec -i $(docker ps -qf name=db) mysql -uroot -pexample < data/schema_add2.sql
docker exec -i $(docker ps -qf name=db) mysql -uroot -pexample < data/schema_add3.sql
```
