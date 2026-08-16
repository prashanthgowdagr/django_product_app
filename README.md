# Product catalog — Django + Postgres + Docker

A minimal Django app with a server-rendered frontend (plain HTML templates,
no JS framework), backed by PostgreSQL, fully containerized. Tested end to
end against a real Postgres instance before packaging.

## What's inside

```
django_product_app/
├── docker-compose.yml     # web (Django) + db (Postgres) services
├── Dockerfile             # builds the Django image
├── requirements.txt       # Django 5.2.17, psycopg2-binary 2.9.10
├── .env.example           # copy to .env
├── manage.py
├── config/                # project settings, urls, wsgi/asgi
└── products/               # the app: model, views, forms, templates
    ├── models.py           # Product (name, description, price, created_at)
    ├── views.py            # list / create / delete
    ├── forms.py            # ProductForm
    ├── admin.py            # registered in Django admin
    └── templates/products/ # product_list.html, product_form.html
```

## Run it locally with Docker

1. Copy the env file:
   ```
   cp .env.example .env
   ```

2. Build and start both containers:
   ```
   docker compose up --build
   ```
   This starts Postgres first, waits until it's healthy, then runs
   `migrate` and starts the Django dev server — you'll see both services'
   logs interleaved in the terminal.

3. Open the app:
   - Product list: http://localhost:8000/
   - Add a product: http://localhost:8000/products/new/
   - Django admin: http://localhost:8000/admin/ (create a superuser first,
     see below)

4. To create an admin login:
   ```
   docker compose exec web python manage.py createsuperuser
   ```

5. To stop everything:
   ```
   docker compose down
   ```
   Add `-v` to also delete the Postgres data volume (`docker compose down -v`).

## How the pieces connect

- `web` container runs Django on port 8000, reading DB connection details from
  environment variables (`POSTGRES_HOST=db` — the service name, since Docker
  Compose gives each service DNS resolution by its name on the shared network).
- `db` container runs Postgres 16, with a named volume (`pgdata`) so your data
  survives `docker compose restart`.
- `depends_on` + a Postgres healthcheck means the web container won't try to
  migrate before the database is actually ready to accept connections.
- The frontend here is Django's own template engine (server-rendered HTML) —
  no separate frontend build step, no API layer. The view queries the ORM and
  renders HTML directly. This is the simplest possible version of the full
  stack: browser → Django view → ORM → Postgres → HTML back.

## Next steps from here

- Swap the plain HTML forms for a JSON API (Django REST Framework) if you
  want to separate the frontend from the backend later.
- Add a `Dockerfile` healthcheck and switch `runserver` for `gunicorn` before
  this touches anything resembling production.
- Add the CI/CD, Helm chart, and ingress layers on top once this runs
  cleanly on your machine — same pattern as your other k3s/EKS work.
