# URL Shortener

A full-stack URL shortener with a React dashboard and a Spring Boot API. Users can register and sign in, create short links, view their links, and inspect click analytics. Visiting a short link redirects to its destination and records a click.

## Repository layout

- `client/` — React 18 frontend built with Vite, React Router, and Tailwind CSS.
- `server/` — Spring Boot 3.5 REST API using Spring Security, JWT, Spring Data JPA, and MySQL.

## Requirements

- Java 23
- Node.js 20 or newer and npm
- Docker Compose

## Run the backend and MySQL

Docker Compose starts MySQL and the Spring Boot API. From the repository root, run:

```sh
cd infra
cp .env.example .env
# Edit .env and set strong values for DATABASE_PASSWORD, MYSQL_ROOT_PASSWORD, and JWT_SECRET.
docker compose up --build
```

The API is available at `http://localhost:8080`; MySQL is published on port `3306`. Compose stores database files in the `mysql_data` volume.

To run the backend outside Docker, start a MySQL database and set `DATABASE_URL` (for example, `jdbc:mysql://localhost:3306/url_shortener`), `DATABASE_USERNAME`, `DATABASE_PASSWORD`, `JWT_SECRET`, and `FRONTEND_URL`, then run `./mvnw spring-boot:run` from `server/`.

## Configure and run the frontend

Set `VITE_BACKEND_URL` to the backend origin. For local development, create `client/.env.local`:

```dotenv
VITE_BACKEND_URL=http://localhost:8080
```

Then run:

```sh
cd client
npm install
npm run dev
```

Vite prints the local development URL when it starts. To create a production build, run `npm run build` from `client/`.

## Features

- Account registration and login with JWT authentication.
- Create and manage short URLs from a private dashboard.
- Public short-link redirects with click tracking.
- Per-link analytics and total click summaries over date ranges.

## API overview

Authentication endpoints are public:

- `POST /api/auth/public/register` — body: `{"username":"...","email":"...","password":"..."}`
- `POST /api/auth/public/login` — body: `{"username":"...","password":"..."}`; returns a JWT token.

The following URL endpoints require the JWT in an `Authorization: Bearer <token>` header:

- `POST /api/urls/shorten` — body: `{"originalUrl":"https://example.com"}`
- `GET /api/urls/myurls`
- `GET /api/urls/analytics/{shortUrl}?startDate={ISO_LOCAL_DATE_TIME}&endDate={ISO_LOCAL_DATE_TIME}`
- `GET /api/urls/totalClicks?startDate={YYYY-MM-DD}&endDate={YYYY-MM-DD}`

A short code can be visited with `GET /{shortUrl}` on the backend host. It responds with a redirect when the code exists, or `404` otherwise.
