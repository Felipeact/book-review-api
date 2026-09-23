# Book Review API

This project is a small Express.js API for a bookshop review application. It exposes a public catalog and a protected customer area for managing book reviews.

## Features

- view all books
- search books by ISBN, author, or title
- register a new customer
- log in as a registered user
- add, update, or delete a review for a specific book

The application stores users and reviews in memory while the Node.js process is running. There is no database or external API dependency.

## Tech stack

- Node.js
- Express
- express-session
- jsonwebtoken
- nodemon

## Project structure

```text
final_project/
+-- index.js
+-- package.json
+-- README.md
+-- router/
    +-- auth_users.js
    +-- booksdb.js
    +-- general.js
```

## Install and run

From the repository root:

```bash
cd final_project
npm install
npm start
```

The app listens on:

```text
http://localhost:8800
```

To run without nodemon:

```bash
cd final_project
node index.js
```

## Important notes

- the app uses sessions to track customer login state
- protected review routes are guarded by JWT validation
- the secret values are hard-coded for local demo use
- there is no automated test suite in this project yet

## Public routes

### Register a user

```http
POST /register
Content-Type: application/json
```

Example:

```bash
curl -X POST http://localhost:8800/register \
  -H "Content-Type: application/json" \
  -d '{"username":"alice","password":"secret"}'
```

### List all books

```http
GET /
```

Example:

```bash
curl http://localhost:8800/
```

### Get a book by ISBN

```http
GET /isbn/:isbn
```

Example:

```bash
curl http://localhost:8800/isbn/1
```

### Get books by author

```http
GET /author/:author
```

Example:

```bash
curl "http://localhost:8800/author/Jane%20Austen"
```

### Get books by title

```http
GET /title/:title
```

Example:

```bash
curl "http://localhost:8800/title/Pride%20and%20Prejudice"
```

### Get reviews for a book

```http
GET /review/:isbn
```

Example:

```bash
curl http://localhost:8800/review/1
```

## Protected customer routes

All authenticated endpoints are mounted under `/customer` and require a valid login session.

### Log in

```http
POST /customer/login
Content-Type: application/json
```

Example:

```bash
curl -c cookies.txt -X POST http://localhost:8800/customer/login \
  -H "Content-Type: application/json" \
  -d '{"username":"alice","password":"secret"}'
```

### Add or update a review

```http
PUT /customer/auth/review/:isbn?review=value
```

Example:

```bash
curl -b cookies.txt -X PUT "http://localhost:8800/customer/auth/review/1?review=A%20great%20classic"
```

### Delete a review

```http
DELETE /customer/auth/review/:isbn
```

Example:

```bash
curl -b cookies.txt -X DELETE http://localhost:8800/customer/auth/review/1
```

## Response behavior

- valid registration: `200 OK`
- duplicate user: `404 Not Found`
- invalid login: `208` with a message
- unauthenticated access: `403 Forbidden`
- missing or invalid book: `404 Not Found`

## Notes for local use

This repo is intended as a learning project. Register a user first, log in, then use the session cookie from the login response when calling the protected review endpoints.
