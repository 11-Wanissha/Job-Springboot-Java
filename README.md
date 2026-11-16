# Job Application REST API

A REST API for a simple job board, built with Spring Boot. It manages **companies**, **jobs** and **company reviews**, with create, read, update and delete endpoints for each.

## Tech Stack

| Area | Technology |
| --- | --- |
| Language | Java 21 |
| Framework | Spring Boot 3.3.5 (Spring Web) |
| Persistence | Spring Data JPA (Hibernate) |
| Database | H2 (in-memory) |
| Monitoring | Spring Boot Actuator |
| Build tool | Maven (Maven Wrapper included) |

## Project Structure

Code is organized by feature (`job`, `company`, `review`). Each feature has an entity, a controller, a service interface with its implementation, and a repository.

```
firstJobApp/src/main/java/com/example/firstJobApp/
├── job/       Job, JobController, JobService, JobRepository, impl/JobServiceImpl
├── company/   Company, CompanyController, CompanyService, CompanyRepository, impl/CompanyServiceImpl
└── review/    Review, ReviewController, ReviewService, ReviewRepository, impl/ReviewServiceImpl
```

## Data Model

- **Company**: `id`, `name`, `description`, `jobs`, `reviews`
- **Job**: `id`, `title`, `description`, `minSalary`, `maxSalary`, `location`, `company`
- **Review**: `id`, `title`, `description`, `rating`, `company`

A company has many jobs and many reviews. In JSON, a company's `jobs` and a review's `company` are hidden (`@JsonIgnore`).

## Getting Started

Requires JDK 21.

```bash
cd firstJobApp
./mvnw spring-boot:run
```

On Windows, use `mvnw.cmd spring-boot:run`. The API runs on `http://localhost:8080`.

## API Endpoints

### Companies

| Method | Endpoint | Description |
| --- | --- | --- |
| GET | `/companies` | List all companies |
| GET | `/companies/{id}` | Get a company |
| POST | `/companies` | Create a company |
| PUT | `/companies/{id}` | Update name, description and reviews |
| DELETE | `/companies/{id}` | Delete a company |

### Jobs

| Method | Endpoint | Description |
| --- | --- | --- |
| GET | `/jobs` | List all jobs |
| GET | `/jobs/{id}` | Get a job |
| POST | `/jobs` | Create a job |
| PUT | `/jobs/{id}` | Update title, description, salary range and location |
| DELETE | `/jobs/{id}` | Delete a job |

### Reviews

| Method | Endpoint | Description |
| --- | --- | --- |
| GET | `/companies/{id}/reviews` | List reviews of a company |
| GET | `/companies/{id}/reviews/{reviewId}` | Get a review |
| POST | `/companies/{id}/reviews` | Add a review (company must exist) |
| PUT | `/companies/{id}/reviews/{reviewId}` | Update a review |
| DELETE | `/companies/{id}/reviews/{reviewId}` | Delete a review |

Responses use `200 OK`, `201 Created` or `404 Not Found`.

### Example

```bash
curl -X POST http://localhost:8080/jobs \
  -H "Content-Type: application/json" \
  -d '{"title": "Software Developer", "description": "Web development", "minSalary": "25000", "maxSalary": "40000", "location": "Bangkok", "company": {"id": 1}}'
```

## Configuration

From `application.properties`:

- H2 in-memory database (`jdbc:h2:mem:test`) with `ddl-auto=create-drop`; SQL is logged to the console.
- H2 console enabled at `/h2-console`.
- All Actuator endpoints exposed (for example `/actuator/health`, `/actuator/info`), the `shutdown` endpoint enabled, and graceful shutdown on.

## Limitations and Cautions

- **Data is not persistent.** The in-memory H2 database is wiped on every restart.
- **Tests:** the project only has the default `contextLoads()` test, which checks that the application starts. Endpoints and business logic are not tested.
- **No security.** There is no authentication or authorization, and the Actuator `shutdown` endpoint and H2 console are open. Use for local development only.
- **No input validation or global error handling.** Invalid or missing fields are not checked, and errors are not mapped to custom responses.
- **Salaries are stored as `String`**, not numbers.
- **Partial updates differ by resource.** Job and company updates skip `null` fields, but a review update replaces the whole review, so omitted fields become `null`.
- **Updates are limited.** Updating a job does not change its company, and updating a company does not change its jobs.
- **Relationship checks are minimal.** Creating a job does not verify that the given company exists, and `GET /companies/{id}/reviews` returns an empty list for an unknown company.
- **Deleting a company** that still has jobs or reviews has no cascade rules and may fail on the foreign key constraint. This was not tested.
