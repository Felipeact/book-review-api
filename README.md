# Express Book Reviews API

This repository contains a small Express.js application for a bookshop review API. The implementation lives under the `final_project/` directory and exposes a public catalog API plus customer-only review endpoints. It is a learning/demo project that uses in-memory data instead of a database.

## What the project does

The application allows a developer to:

- browse the available book catalog
- search books by ISBN, author, or title
- register a username/password pair
- log in as a registered customer
- add, update, or delete a review for a specific book

The app does not use a database, a cache, or external APIs for book data. Users and reviews are stored in memory while the Node.js process is running.

## Architecture overview

The application is structured around a minimal Express server and two route modules:

- `final_project/index.js` starts the Express app, configures middleware, and mounts the routers
- `final_project/router/general.js` exposes the public routes for browsing and registration
- `final_project/router/auth_users.js` contains login and authenticated review routes
- `final_project/router/booksdb.js` stores the in-memory book catalog

The server uses:

- `express-session` to manage a session for customer requests
- `jsonwebtoken` to sign and verify a bearer-style access token stored in the session
- `axios` inside the public router to call the local API endpoints when populating book data

## Technology stack

The project dependencies declared in `final_project/package.json` are:

- `express` for the HTTP server and routing
- `express-session` for session handling
- `jsonwebtoken` for JWT creation and verification
- `axios` as a dev dependency used by the router code
- `nodemon` for local development restarts

There is no database layer, ORM, or framework such as Sequelize, Mongoose, or Prisma in this repository.

## Project structure

```text
expressBookReviews/
├── LICENSE
├── README.md
├── cookies.txt
└── final_project/
    ├── index.js
    ├── package.json
    ├── README.md
    └── router/
        ├── auth_users.js
        ├── booksdb.js
        └── general.js
```

## Prerequisites

You need:

- Node.js and npm installed on your machine
- a terminal with access to the workspace

This repository does not include a `.env` file, Docker configuration, or deployment manifest.

## Configuration

This project does not use environment variables.

The runtime configuration is hard-coded in source files:

- `final_project/index.js` sets the listening port to `8800`
- `app.use("/customer", session({ secret: "fingerprint_customer", resave: true, saveUninitialized: true }))`
- JWT secret is set to `"access"` in `final_project/router/auth_users.js`

Because the secrets are embedded in code, this is intended for local learning/demo use only.

## Setup and installation

From the repository root:

```bash
cd final_project
npm install
```

This installs the dependencies declared in `final_project/package.json`.

## How to run the server

Start the API with:

```bash
cd final_project
npm start
```

The server starts with `nodemon index.js` and logs:

```text
Server is running
```

The app listens on:

```text
http://localhost:8800
```

## Running the app without nodemon

The project can also be started directly with Node.js if you want to bypass the file watcher:

```bash
cd final_project
node index.js
```

## Testing and validation

There is no automated test suite in this repository.

The `final_project/package.json` currently defines:

```json
"test": "echo \"Error: no test specified\" && exit 1"
```

So `npm test` is not a valid test command for this codebase. Manual validation is performed with HTTP requests using `curl`.

## Public API endpoints

The following routes are mounted at the application root (`/`).

### 1. Register a user

```http
POST /register
Content-Type: application/json
```

Request body:

```json
{
  "username": "alice",
  "password": "secret"
}
```

Behavior:

- validates that both fields are present
- rejects duplicates with a `404` response if the user already exists
- stores the new user in the in-memory `users` array

Example:

```bash
curl -X POST http://localhost:8800/register \
  -H "Content-Type: application/json" \
  -d '{"username":"alice","password":"secret"}'
```

### 2. Get all books

```http
GET /
```

Returns the current in-memory book catalog.

Example:

```bash
curl http://localhost:8800/
```

### 3. Get a book by ISBN

```http
GET /isbn/:isbn
```

Example:

```bash
curl http://localhost:8800/isbn/1
```

### 4. Get books by author

```http
GET /author/:author
```

Example:

```bash
curl "http://localhost:8800/author/Jane%20Austen"
```

### 5. Get books by title

```http
GET /title/:title
```

Example:

```bash
curl "http://localhost:8800/title/Pride%20and%20Prejudice"
```

### 6. Get reviews for a book

```http
GET /review/:isbn
```

Example:

```bash
curl http://localhost:8800/review/1
```

## Customer authentication and protected routes

The protected customer routes are mounted under `/customer` and require a valid session created by login.

### 1. Log in

```http
POST /customer/login
Content-Type: application/json
```

Request body:

```json
{
  "username": "alice",
  "password": "secret"
}
```

Behavior:

- checks the username/password pair against the in-memory user list
- signs a JWT using the secret `"access"`
- stores the token and username in `req.session.authorization`
- responds with success when authentication succeeds

Example:

```bash
curl -c cookies.txt -X POST http://localhost:8800/customer/login \
  -H "Content-Type: application/json" \
  -d '{"username":"alice","password":"secret"}'
```

The session cookie is created by `express-session` and is required for subsequent protected requests.

### 2. Add or update a review

```http
PUT /customer/auth/review/:isbn?review=your-review
```

The route validates:

- the JWT in `req.session.authorization.accessToken`
- that the book exists in the in-memory catalog
- that the `review` query parameter is provided

Example:

```bash
curl -b cookies.txt -X PUT "http://localhost:8800/customer/auth/review/1?review=Excellent%20classic" \
  -H "Content-Type: application/json"
```

This stores the review under the logged-in username.

### 3. Delete a review

```http
DELETE /customer/auth/review/:isbn
```

Example:

```bash
curl -b cookies.txt -X DELETE http://localhost:8800/customer/auth/review/1
```

This removes the current user’s review for the given book.

## Authentication behavior

The middleware in `final_project/index.js` enforces authentication for all requests under `/customer/auth/*`:

- it checks whether `req.session.authorization` exists
- reads `req.session.authorization.accessToken`
- verifies the JWT with the secret `"access"`
- rejects the request with `403` if the token is missing or invalid

Example failure response:

```json
{ "message": "User not authenticated" }
```

## In-memory data model

The catalog is defined in `final_project/router/booksdb.js` and contains ten sample books.

Each book record has this shape:

```js
{
  author: "Jane Austen",
  title: "Pride and Prejudice",
  reviews: {}
}
```

The user registry is defined in `final_project/router/auth_users.js` as an array:

```js
let users = [];
```

Reviews are stored per book as an object keyed by the review author username:

```js
books[isbn].reviews[username] = review;
```

## Notable implementation details and limitations

This project is intentionally simple but a few important behaviors should be known:

- there is no persistent database; restarting the server resets all users and reviews
- the app relies on hard-coded secrets and in-memory state
- there is no `.env` support or configuration management
- there is no automated test suite
- the project does not include Docker, Kubernetes, or CI/CD configuration
- the public router triggers internal `axios` calls on the same local server to fetch catalog data

## Deployment guidance

There is no production deployment configuration in the repository.

To deploy this app to a host or server:

1. copy the `final_project/` directory to the target machine
2. install dependencies with `npm install`
3. run the app with `node index.js` or `npm start`
4. expose port `8800` to the outside world or use a proxy such as NGINX or a reverse proxy in front of Node.js

Because the application stores data in memory and uses hard-coded secrets, it is best suited for local development, demos, and learning environments rather than a production-grade service.

## Quick validation checklist

Use the following flow to verify the app behavior manually:

```bash
cd final_project
npm install
npm start
```

In another terminal:

```bash
curl http://localhost:8800/
curl -X POST http://localhost:8800/register -H "Content-Type: application/json" -d '{"username":"alice","password":"secret"}'
curl -c cookies.txt -X POST http://localhost:8800/customer/login -H "Content-Type: application/json" -d '{"username":"alice","password":"secret"}'
curl -b cookies.txt -X PUT "http://localhost:8800/customer/auth/review/1?review=Great%20book"
curl -b cookies.txt -X DELETE http://localhost:8800/customer/auth/review/1
```

## Summary

This repository is a lightweight Express-based book review API built for learning and demonstration. Its core strengths are simplicity, clear route organization, and moderate use of session/JWT-based access control. Its main limitations are the lack of persistence, hard-coded configuration, and absence of formal tests or deployment automation.
