# Books API

Lightweight Spring Boot application (Java 21) that exposes a small Books REST API and a minimal Thymeleaf web UI for searching a catalog of books.

## Features
- Loads book data from a configurable remote JSON (`books.remote.url`) with automatic fallback to `classpath:books.json`.
- In-memory cache with a manual reload endpoint.
- REST search endpoints:
  - free-text search (single query)
  - multi-keyword search with ANY/ALL match modes
- Minimal, responsive Thymeleaf UI for basic and advanced searches.
- Unit tests for service and controller behavior.

## Architecture
This is a simple layered Spring Boot application:
- **Controller Layer**: `BooksController` (REST API) and `BooksWebController` (Thymeleaf web views)
- **Service Layer**: `BooksService` handles data loading, caching, and search logic
- **Model Layer**: `Book` entity with Lombok annotations
- **Data Access**: Remote HTTP fetch with classpath fallback, parsed via Jackson
- **View Layer**: Thymeleaf templates for the web UI

## Quick start
Requirements: Java 21, or use the bundled Maven wrapper.

From project root (Windows PowerShell):

    .\mvnw.cmd clean package
    .\mvnw.cmd spring-boot:run

App runs on http://localhost:8080 by default.

## Configuration
Edit `src/main/resources/application.properties` or override at runtime.
- `books.remote.url` — URL to a remote JSON list of books (optional). If fetching fails the bundled `books.json` is used.

## REST API
Base path: `/api/books`

- GET `/api/books?query={q}`
  - Free-text search across title, subtitle, author, description, publisher, isbn. Omit `query` to return all books.

- POST `/api/books/search`
  - JSON body: `{ "keywords": ["k1","k2"], "matchMode": "ANY|ALL" }`
  - `matchMode` = `ALL` requires all keywords; otherwise ANY behavior is used.

- GET `/api/books/reload`
  - Reloads source (attempts remote then classpath) and returns `{ count, lastLoaded }`.

- GET `/api/books/health`
  - Returns `{ "status": "UP" }`.

## Web UI
- GET `/` — Home page with:
  - Basic search (single query)
  - Advanced search (comma-separated keywords + ANY/ALL selection)
- POST `/search` — Basic web search
- POST `/search/advanced` — Advanced web search
- GET `/reload` — Reload data and show metadata on the page

The Thymeleaf template is at `src/main/resources/templates/books.html`.

## Examples

List all books:

    curl http://localhost:8080/api/books

Single keyword search:

    curl "http://localhost:8080/api/books?query=javascript"

Multi-keyword ANY search:

    curl -X POST -H "Content-Type: application/json" -d '{"keywords":["modern","press"],"matchMode":"ANY"}' \
      http://localhost:8080/api/books/search

# Multi-keyword ALL
curl -H "Content-Type: application/json" -d '{"keywords":["javascript","introduction"],"matchMode":"ALL"}' \
  http://localhost:8080/api/books/search
```

## Project Structure
```
src/
  main/
    java/com/example/cognizant/Books/
      BooksApplication.java
      controller/BooksController.java
      service/BooksService.java
      model/Book.java
    resources/
      application.properties
      books.json (fallback)
  test/
    java/com/example/cognizant/Books/
      BooksApplicationTests.java
      controller/BooksControllerTest.java
      service/BooksServiceTest.java
postman_collection.json
postman_environment.json
README.md
HELP.md
```

## Extending / Next Steps
- Pagination & sorting (e.g., by published date or title).
- ETag / conditional GET for client-side caching.
- Scheduled refresh or cache TTL invalidation.
- Add OpenAPI/Swagger via `springdoc-openapi` dependency.
- Validation & error responses for malformed POST bodies.
- Metrics & tracing integration (Micrometer / OpenTelemetry).
- Containerization (Dockerfile + CI pipeline).

## Using GitHub Actions (CI)

This project includes a GitHub Actions workflow that builds and tests the project automatically.

How to run it:

1. Push a commit to the `main` branch or open a pull request — the workflow will run automatically.
2. Or trigger it manually from the GitHub UI: go to the "Actions" tab, select the workflow named `CI`, then click "Run workflow".

What the workflow does:

- Checks out your repository
- Sets up Java 21
- Uses the Maven wrapper (`mvnw` / `mvnw.cmd`) to build and run tests
- Caches Maven dependencies to speed up subsequent runs

How to view results:

1. On GitHub, open the "Actions" tab and choose the most recent run. Steps and logs are visible there.
2. If a step fails, expand it to see the console output. The error message usually points to the failing test or build issue.

Optional: GitHub Badge

You can add a status badge to this README to show the build status. After the workflow runs, copy the badge from the workflow page and paste it at the top of this file.

If you'd like, I can add the badge automatically once you confirm the workflow runs on GitHub. 

---
Feel free to adapt or extend. PRs / improvements welcome.
