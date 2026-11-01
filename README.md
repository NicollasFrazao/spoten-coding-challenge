# Spoten Back-end Challenge

A small Laravel 8 JSON API for a movie catalog with user comments (ratings).

## Context

Solution to the back-end hiring challenge ("Desafio Back-end") of Spoten (2021).

## What it does

- `movie` table: name, year, synopsis (`sinopse`), duration, directors, writers, cast (`stars`, JSON), score (`rating`, decimal) and poster image URL.
- `rating` table: a free-text `comment` linked to a movie (`movie_id`, foreign key with cascade on update/delete).
- A seeder loads the eight Harry Potter films (Portuguese titles and synopses).
- Endpoints, registered as resource routes in `routes/web.php` (CSRF verification is disabled for all routes):

| Method | Path | Behavior |
| --- | --- | --- |
| GET | `/movies` | All movies, each with its ratings |
| GET | `/movies/{id}` | The movie with that id and its ratings (returned as a one-element array) |
| GET | `/ratings` | All ratings, each with its movie |
| GET | `/ratings/{id}` | The rating with that id and its movie (one-element array) |
| POST | `/ratings` | Creates a rating from `comment` and `movie_id`; returns `{ "success": bool, "message": string }` |

The remaining resource actions (create/store for movies, edit, update, destroy) exist as empty stubs.

## Tech stack

- PHP `^7.3|^8.0`
- Laravel Framework 8 (`^8.12`, locked at v8.22.1)
- MySQL (Eloquent models use a dedicated `spoten` connection)
- PHPUnit 9 (locked at 9.5.0)
- Laravel Sail 1.x (`docker-compose.yml`: PHP 8.0 runtime, MySQL 8.0, Redis, MailHog)
- Laravel Mix 6 for front-end assets (not used by the API)

## Running locally

Without Docker (PHP, Composer and MySQL installed):

```bash
composer install
cp .env.example .env
php artisan key:generate
# set DB_HOST, DB_DATABASE, DB_USERNAME and DB_PASSWORD in .env
php artisan migrate --seed
php artisan serve
```

Then open `http://localhost:8000/movies`.

With Laravel Sail (the compose file builds from `vendor/`, so run `composer install` first):

```bash
composer install
cp .env.example .env    # set DB_HOST=mysql and a DB_PASSWORD
./vendor/bin/sail up -d
./vendor/bin/sail artisan key:generate
./vendor/bin/sail artisan migrate --seed
```

Example request:

```bash
curl -X POST http://localhost:8000/ratings -d comment="Great movie" -d movie_id=1
```

## Tests

```bash
php artisan test
```

Only the default Laravel example tests are present (`tests/Feature/ExampleTest.php` checks that `/` returns 200).

## Project structure

```
app/Http/Controllers/   MovieController, RatingController
app/Models/             Movie, Rating (and the default User)
database/migrations/    movie and rating tables
database/seeders/       MovieSeeder (Harry Potter films)
routes/web.php          resource routes for movies and ratings
config/database.php     extra "spoten" MySQL connection
```

## Notes

- No dedicated tests for the movie/rating endpoints.
- The endpoints live in `routes/web.php` rather than `routes/api.php`, with CSRF disabled for every route (`VerifyCsrfToken::$except = ['*']`).
- `POST /ratings` only checks that `comment` and `movie_id` are present; it does not validate that the movie exists, and an unknown `movie_id` surfaces as a database foreign-key error.
- `show` endpoints return an array instead of a single object and do not return 404 for unknown ids.
- There is no authentication.
- The `spoten` connection falls back to `root`/`root` when the database variables are not set.
