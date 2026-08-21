website link
https://v26r5tzydt.konigle.net/


Luxury Spa — Django Booking Backend

A Django-based backend for the Luxury Spa website. It provides the booking API, availability checking, waitlist handling, Django Admin dashboard, Contact Us message processing, Cloudflare Turnstile verification, and an optional Gemini/Google ADK booking agent.

Features

Appointment booking API

Customer name, email, phone, service, date, time, therapist preference and notes

Automatic booking reference IDs such as SPA-2026-1234

Input validation and privacy-policy consent checking

Optional API-key protection

Availability management

Configurable recurring daily time slots

Active/inactive spa services

Date-specific availability checks

Confirmed bookings automatically remove a slot from availability for that date

Automatic waitlist

If a requested date/time already has a confirmed booking, the new request is stored as WAITLISTED

When a confirmed booking is cancelled, the earliest waitlisted booking for that same slot can be promoted automatically

Django Admin dashboard

View, search and filter bookings

Update booking status

Manage services and time slots

View Contact Us messages

See the agent's classification and reply

Export selected bookings to CSV

Contact Us workflow

Saves customer name, email and message in ContactMessage

Classifies messages such as rescheduling requests

Matches a reschedule request using both customer name and email

Checks the real booking calendar

Reschedules when the requested slot is available

Moves the booking to the waitlist when the requested slot is full

Stores the agent's response and affected booking in the database

Cloudflare Turnstile

Optional bot protection for public booking submissions

Secret key is read from environment variables rather than source code

Gemini / Google ADK agent

Checks services and prices

Checks availability

Books appointments

Cancels appointments

Reschedules appointments

Views bookings and waitlist

Processes Contact Us messages

Uses the same Django database as the website

Project Structure

spa_backend/
├── manage.py
├── requirements.txt
├── .env
├── bookings/
│   ├── admin.py
│   ├── apps.py
│   ├── models.py
│   ├── services.py
│   ├── views.py
│   ├── urls.py
│   ├── middleware.py
│   └── ...
├── my_agent/
│   ├── __init__.py
│   └── agent.py
├── spabackend/
│   ├── settings.py
│   ├── urls.py
│   ├── asgi.py
│   └── wsgi.py
├── templates/
├── static/
└── staticfiles/

Requirements

Python 3.10+

Django 5.2+

A virtual environment is recommended

Google Gemini / Google ADK only if the agent functionality is required

PostgreSQL can be used through DATABASE_URL; SQLite is used by default

Install the dependencies with:

pip install -r requirements.txt

Configuration

Create a .env file in the project root.

Example:

DJANGO_SECRET_KEY=replace-with-a-new-secret-key
DJANGO_DEBUG=true
DJANGO_TIME_ZONE=Asia/Singapore

FRONTEND_URL=https://your-website.example/

REQUIRE_CAPTCHA=false
TURNSTILE_SITEKEY=
TURNSTILE_SECRET=

AGENT_API_KEY=

CORS_ALLOWED_ORIGINS=http://localhost:3000

Important security warning

Do not commit .env to GitHub.

The .env file may contain:

Django secret keys

Cloudflare Turnstile secrets

API keys

Email credentials

If a secret has already been committed to a public repository, treat it as compromised and replace/rotate it.

For production:

Set DJANGO_DEBUG=false

Generate a new DJANGO_SECRET_KEY

Set a specific ALLOWED_HOSTS value

Restrict CORS_ALLOWED_ORIGINS to the real frontend

Configure CSRF_TRUSTED_ORIGINS appropriately

Use a production database such as PostgreSQL

Configure real email credentials

Enable Turnstile with a production secret

Database

The default database is SQLite:

db.sqlite3

The application can also use another database through DATABASE_URL.

For example:

DATABASE_URL=postgresql://USER:PASSWORD@HOST:5432/DATABASE

After installing dependencies, run:

python manage.py migrate

Create an administrator account with:

python manage.py createsuperuser

Then start Django:

python manage.py runserver

The local backend is normally available at:

http://127.0.0.1:8000/

The Django Admin is available at:

http://127.0.0.1:8000/admin/

API Endpoints

Check backend status

GET /api/status/

Returns information about the backend, CAPTCHA configuration and booking count.

Get services and recurring time slots

GET /api/services/

Example response:

{
  "services": {
    "swedish": {
      "name": "Swedish Massage",
      "duration": "60 min",
      "price": 120.0
    }
  },
  "time_slots": ["09:00", "10:00", "11:00"]
}

Check availability for a date

GET /api/availability/?date=2026-08-25

Also available as:

GET /api/available-times/?date=2026-08-25

Create a booking

POST /api/book/

The booking request can contain:

{
  "fullName": "Jane Doe",
  "email": "jane@example.com",
  "phone": "91234567",
  "service": "swedish",
  "date": "2026-08-25",
  "time": "14:00",
  "therapist": "No preference",
  "notes": "First visit",
  "consent": true,
  "cf-turnstile-response": "TURNSTILE_TOKEN"
}

If the requested slot is available, the booking is created as CONFIRMED.

If the slot already has a confirmed booking, the new booking is created as WAITLISTED.

Submit a Contact Us message

POST /api/contact/

Example:

{
  "name": "Jane Doe",
  "email": "jane@example.com",
  "message": "I cannot make my appointment. Could I change it to 25 August at 2pm?"
}

The message is stored as a ContactMessage and passed through the contact-message processing workflow.

Booking Statuses

Bookings can have the following statuses:

Status

Meaning

PENDING

Booking is awaiting processing

CONFIRMED

Appointment is confirmed

COMPLETED

Appointment has been completed

CANCELLED

Appointment has been cancelled

NO_SHOW

Customer did not attend

WAITLISTED

Requested slot was full

Contact Message Statuses

Status

Meaning

PENDING

Message has been received

PROCESSING

Agent is processing the request

PROCESSED

Request has been handled

WAITING_FOR_CUSTOMER

More information is required

FAILED

The requested action could not be completed

Rescheduling Workflow

The Contact Us system is designed to handle messages such as:

"Hi, I won't be able to make my appointment. Could I change my booking date?"

The workflow is:

Customer submits Contact Us form
              ↓
ContactMessage is created
              ↓
Agent classifies the message
              ↓
Is it a reschedule request?
        ┌─────┴─────┐
       No           Yes
       ↓             ↓
General reply   Extract new date/time
                      ↓
             Match name + email
                      ↓
              Find existing booking
                      ↓
             Check requested slot
                ┌─────┴─────┐
             Available      Full
                 ↓            ↓
            Reschedule     Waitlist
                 └─────┬──────┘
                       ↓
                Save agent reply
                       ↓
                Notification email

If the customer provides only a new time, the existing booking date is retained.

If the customer provides only a new date, the existing booking time is retained.

Django Admin

Open:

http://127.0.0.1:8000/admin/

The admin dashboard can be used to:

Manage bookings

Manage services

Open or close recurring time slots

View Contact Us messages

See booking statuses

See which booking a Contact Us message affected

Re-run Contact Us processing

Export bookings to CSV

Google ADK Agent

The optional my_agent/ application provides a Gemini-powered booking agent.

The agent uses the same Django ORM and database as the website. There is no separate booking database for the agent.

This means:

Website booking
      ↓
Django ORM
      ↓
Same database
      ↑
Django Admin
      ↑
Gemini Agent

The agent can work with:

Services and prices

Availability

New bookings

Cancellations

Rescheduling

Waitlists

Booking lists

Contact Us messages

Running the agent

The project is designed so Django and the ADK web interface do not compete for the same default port.

Run Django on port 8000:

python manage.py runserver 8000

Run the agent on port 8001 using the project's ADK entry point/configuration:

adk web --port 8001

The exact ADK command may vary with the installed Google ADK version.

Async Django ORM handling

The agent runs in an asynchronous environment. Django ORM operations are synchronous by default.

To prevent:

SynchronousOnlyOperation:
You cannot call this from an async context

the agent wraps database operations in synchronous helper functions and calls them through sync_to_async.

This is important when running the agent through an async ADK web server.

Email

Email sending is designed to work in two modes.

Development mode

If EMAIL_HOST is empty, customer-facing email content is handled as a development/dummy notification and is recorded so it can be inspected through the application.

Production mode

Configure your SMTP provider:

EMAIL_HOST=smtp.example.com
EMAIL_PORT=587
EMAIL_HOST_USER=your-user
EMAIL_HOST_PASSWORD=your-password
EMAIL_USE_TLS=true
DEFAULT_FROM_EMAIL=no-reply@yourdomain.com

No major application-code change should be required for a normal SMTP provider.

Cloudflare Turnstile

Turnstile is optional.

For local testing:

REQUIRE_CAPTCHA=false

For production:

REQUIRE_CAPTCHA=true
TURNSTILE_SITEKEY=your-site-key
TURNSTILE_SECRET=your-secret-key

The secret must remain on the backend and must never be placed in frontend JavaScript.

Frontend Integration

The public frontend can communicate with the backend through the API endpoints.

A typical flow is:

Luxury Spa Website
       ↓
POST /api/book/
       ↓
Django validation
       ↓
Turnstile verification
       ↓
Availability check
       ↓
Create CONFIRMED or WAITLISTED booking
       ↓
JSON response

The backend also exposes:

/widget.js

for the project's public booking-widget integration.

Production Checklist

Before deploying publicly:

Replace the development Django secret key

Set DJANGO_DEBUG=false

Restrict allowed hosts

Restrict CORS origins

Configure CSRF trusted origins

Rotate any previously exposed API/CAPTCHA secrets

Keep .env out of Git

Configure a production database

Configure real email delivery

Configure Cloudflare Turnstile

Use HTTPS

Create a secure Django admin account

Back up the database

Test booking, cancellation and rescheduling workflows

Test waitlist promotion

Test the Contact Us workflow

Test the frontend against the production API URL

Common Problems

SynchronousOnlyOperation

This usually means synchronous Django ORM code is being called directly from an async context.

The agent should use the existing sync_to_async wrappers instead of calling Django ORM queries directly from an async function.

Port 8000 is already in use

Django and ADK can both default to port 8000.

Use:

python manage.py runserver 8000

and run ADK on another port, such as:

adk web --port 8001

Booking says it succeeded but does not appear in Admin

Check:

The backend is connected to the expected database.

The request returned a successful API response.

The Django migration has been applied.

You are opening the Admin from the same Django instance/database.

The current Django implementation saves bookings through the ORM and returns an error when saving fails.

Turnstile verification fails

Check:

REQUIRE_CAPTCHA

TURNSTILE_SITEKEY

TURNSTILE_SECRET

The frontend is sending cf-turnstile-response

The Turnstile site key matches the website/domain

The secret has not expired or been rotated

Development Notes

The project uses:

Django ORM

Django Admin

SQLite by default

PostgreSQL support through DATABASE_URL

WhiteNoise for static files

Gunicorn for production WSGI serving

Cloudflare Turnstile

Google Gemini / Google ADK for the optional agent

Asia/Singapore as the default timezone

The backend is intended to be the single source of truth for bookings. The website, Django Admin and agent should all operate on the same database rather than maintaining separate copies of booking data.

License

This project is currently a private/custom Luxury Spa backend project. Add an appropriate license here if the repository will be distributed publicly.
