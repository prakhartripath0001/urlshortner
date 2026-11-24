# URL Shortener

A full-stack URL shortener with a React dashboard and a Spring Boot API. Users can register and sign in, create short links, view their links, and inspect click analytics. Visiting a short link redirects to its destination and records a click.

## Repository layout

- `client/` — React 18 frontend built with Vite, React Router, and Tailwind CSS.
- `server/` — Spring Boot 3.4 REST API using Spring Security, JWT, Spring Data JPA, and PostgreSQL.

## Requirements

- Java 23
- Node.js and npm
- PostgreSQL

## Configure the backend

The backend reads its database, JWT, and frontend settings from environment variables. Create a PostgreSQL database, then set:

| Variable | Description |
| --- | --- |
| `DATABASE_URL` | JDBC connection URL, such as `jdbc:postgresql://localhost:5432/url_shortener` |
| `DATABASE_USERNAME` | PostgreSQL username |
| `DATABASE_PASSWORD` | PostgreSQL password |
| `DATABASE_DIALECT` | Hibernate dialect, usually `org.hibernate.dialect.PostgreSQLDialect` |
| `JWT_SECRET` | Secret key used to sign JWTs; provide a sufficiently long, private value |
| `FRONTEND_URL` | Frontend origin allowed by the backend CORS configuration, e.g. `http://localhost:5173` |

Start the API from the repository root:

```sh
cd server
export DATABASE_URL='jdbc:postgresql://localhost:5432/url_shortener'
export DATABASE_USERNAME='postgres'
export DATABASE_PASSWORD='your-password'
export DATABASE_DIALECT='org.hibernate.dialect.PostgreSQLDialect'
export JWT_SECRET='replace-with-a-long-random-secret'
export FRONTEND_URL='http://localhost:5173'
./mvnw spring-boot:run
```

The API runs on port `8080` by default. Hibernate is configured with `ddl-auto=update`, so it updates the schema at startup.

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
