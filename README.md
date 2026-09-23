# Book Review API

This repository contains a small Express.js practice project for a bookshop review API. The implementation lives in the `final_project/` directory and keeps the catalog, users, and reviews in memory while the server is running.

## Project overview

The app supports:

- viewing the book catalog
- searching books by ISBN, author, or title
- registering a new customer
- logging in as a registered user
- adding, updating, or deleting reviews

This is a learning/demo project intended for local use, not a production-ready service with a real database or deployment setup.

## Repository structure

```text
book-review-api/
+-- LICENSE
+-- README.md
+-- final_project/
¦   +-- index.js
¦   +-- package.json
¦   +-- README.md
¦   +-- router/
¦       +-- auth_users.js
¦       +-- booksdb.js
¦       +-- general.js
```

## Quick start

```bash
cd final_project
npm install
npm start
```

The API runs on:

```text
http://localhost:8800
```

## More details

For the full route list, request examples, and authentication flow, see the detailed project guide in [final_project/README.md](final_project/README.md).

## Notes

- No database is used.
- User and review data are stored in memory.
- JWT-based authentication is used for protected routes.
- The app is meant for local learning and testing only.
