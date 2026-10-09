# CampusSlot email notifications (fast request path)

CampusSlot sends transactional email through **Resend's HTTPS API** when `RESEND_API_KEY` is configured. Gmail SMTP is supported as a local-development fallback.

## 1. Local setup

1. Copy `.env.example` to `.env`.
2. Set `RESEND_API_KEY` to your Resend API key.
3. For a quick test, set `RESEND_FROM_EMAIL=CampusSlot <onboarding@resend.dev>`. Resend's shared test sender can only deliver to the email address associated with your Resend account.
4. For real student/teacher recipients, verify a domain in Resend and use a sender address on that verified domain, such as `CampusSlot <notifications@yourdomain.com>`.
5. Set `BASE_URL` to the URL users will open from email. For the same PC, use `http://127.0.0.1:5000`; for Render, use the app's public HTTPS URL.
6. Start the app with `python run.py`.

Never commit `.env`, API keys, or Gmail App Passwords.

## 2. Render setup

In Render → your `campusslot` Web Service → **Environment**, set:

- `RESEND_API_KEY`: your Resend API key (keep it secret).
- `RESEND_FROM_EMAIL`: `CampusSlot <onboarding@resend.dev>` for a limited smoke test, or a sender on your verified domain for production.
- `BASE_URL`: the public HTTPS URL of your deployed CampusSlot app.
- `DATABASE_URL`: keep the existing database connection configured by Render.

Save changes and let Render redeploy. Do not delete/recreate the database.

## 3. Why email won't slow down the buttons

Booking, cancellation, and registration actions enqueue email work instead of waiting for the provider's network response. If `CELERY_BROKER_URL` points to a working Redis broker, CampusSlot uses Celery. If no broker is configured (common on a simple/free Render setup), it uses a small in-process background thread pool.

The in-process fallback keeps button responses fast, but queued messages can be lost if the web process restarts before sending them. For a high-volume production deployment, use a managed Redis broker and a separate Celery worker. Keep the same `CELERY_BROKER_URL` on both the web service and worker.

The administrator's explicit notification test is intentionally synchronous so the admin can see the real send result.

## 4. Verify delivery

- Check `/admin/notifications/status` while logged in as admin to confirm the app sees the email configuration.
- Use the admin dashboard's notification test for an existing appointment.
- Check Render logs for `CampusSlot EMAIL RESULT`, `Background appointment email failed`, or Resend/provider errors.
- A successful API acceptance does not guarantee inbox delivery; check spam and the provider's delivery dashboard.

## 5. Email action links

Action links use the configured `BASE_URL`, are signed, expire after 72 hours, and require a confirmation POST before changing an appointment. `localhost` and `127.0.0.1` point to the device opening the link, not to your deployed app.
