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

Multi-keyword ALL search:

    curl -X POST -H "Content-Type: application/json" -d '{"keywords":["java","spring"],"matchMode":"ALL"}' \
      http://localhost:8080/api/books/search

## Tests
Run unit tests with the Maven wrapper:

    .\mvnw.cmd test

## Project structure (high level)

- `src/main/java/com/example/cognizant/Books/`
  - `BooksApplication.java` — Spring Boot entry point
  - `controller/BooksController.java` — REST endpoints
  - `controller/BooksWebController.java` — Thymeleaf web controller
  - `service/BooksService.java` — loading/parsing/search logic
  - `model/Book.java` — data model (Lombok)
- `src/main/resources/` — `application.properties`, `books.json`, `templates/books.html`
- `pom.xml` — Maven build (includes Thymeleaf dependency)

## Notes / Next steps
- Consider adding pagination, sorting and OpenAPI docs (springdoc).
- Add validation for POST bodies and better error responses.
- Add CI badge after enabling build on GitHub Actions.

---

If you want, I can also add examples for dockerizing the app or an OpenAPI spec file next.
