# API Performance Benchmark & Jira Defect Management Log

## 1. Apache JMeter Load Testing Benchmark (100 Concurrent Virtual Users)

- **Test Tool**: Apache JMeter 5.6.3 (`jmeter/API_Load_Test_100_Concurrent_Users.jmx`)
- **Concurrency**: 100 Virtual Users (Threads) | **Ramp-Up Period**: 30 Seconds | **Loop Count**: 5 (500 Samples per Endpoint)

| Endpoint | HTTP Method | Samples | Avg Response Time (ms) | 90th Percentile (ms) | 95th Percentile (ms) | Throughput (req/sec) | Error Rate (%) | SLA (<2000ms) |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| `/posts` | `GET` | 500 | 318 ms | 540 ms | 675 ms | 44.6 / sec | 0.0% | **PASS** |
| `/posts/1` | `GET` | 500 | 264 ms | 430 ms | 512 ms | 49.2 / sec | 0.0% | **PASS** |
| `/users` | `GET` | 500 | 295 ms | 490 ms | 610 ms | 46.1 / sec | 0.0% | **PASS** |
| `/posts` | `POST` | 500 | 590 ms | 1,120 ms | 1,380 ms | 29.8 / sec | 0.8% | **PASS** |
| `/auth/token` | `POST` | 500 | 1,840 ms | 2,950 ms | 3,420 ms | 13.5 / sec | 4.2% | **FAIL (BUG-104)** |

## 2. Documented Jira Defect Logs

### BUG-101: `POST /orders` accepts negative price and zero quantity without validation error
- **Issue Type**: Bug | **Severity**: High (S2) | **Priority**: P1 | **Status**: Open
- **Environment**: QA Staging API (`v1.4.2`)
- **Steps to Reproduce**:
  1. Open Postman and select request `POST /orders`.
  2. Pass JSON request body `{"userId": 101, "title": "Test Item", "quantity": 0, "price": -49.99}`.
  3. Send request and inspect status code and response payload.
- **Expected Result**: API should return `422 Unprocessable Entity` or `400 Bad Request` rejecting negative prices and zero quantities.
- **Actual Result**: API returns `201 Created` and persists negative price `-49.99`.

### BUG-102: `PUT /posts/{id}` returns `500 Internal Server Error` when `id` exceeds existing range
- **Issue Type**: Bug | **Severity**: High (S2) | **Priority**: P2 | **Status**: Open
- **Steps to Reproduce**:
  1. Send `PUT /posts/99999` with a valid JSON payload.
  2. Observe HTTP status code.
- **Expected Result**: API should return `404 Not Found` for a non-existent resource ID.
- **Actual Result**: Server throws an unhandled exception and returns `500 Internal Server Error`.

### BUG-103: Missing `X-RateLimit-Remaining` and `Retry-After` headers during burst traffic
- **Issue Type**: Bug | **Severity**: Low (S4) | **Priority**: P3 | **Status**: Open
- **Expected Result**: API responses should include standard rate-limit headers (`X-RateLimit-Limit`, `X-RateLimit-Remaining`).
- **Actual Result**: Rate-limit headers are missing across all endpoints.

### BUG-104: `POST /auth/token` 95th percentile response time (3,420ms) breaches 2,000ms SLA under 100-user load
- **Issue Type**: Performance Defect | **Severity**: Critical (S1) | **Priority**: P1 | **Status**: In Progress
- **Steps to Reproduce**:
  1. Run JMeter test plan `jmeter/API_Load_Test_100_Concurrent_Users.jmx` with 100 threads and 30s ramp-up.
  2. Inspect the Aggregate Report for `POST /auth/token`.
- **Expected Result**: 95th percentile latency `< 2000ms` and error rate `< 1.0%`.
- **Actual Result**: 95th percentile latency reached `3,420ms` with `4.2%` HTTP `503 Service Unavailable` timeouts due to synchronous bcrypt hashing blocking worker threads.

### BUG-105: `DELETE /posts/{id}` returns `200 OK` with empty object `{}` for already-deleted IDs
- **Issue Type**: Bug | **Severity**: Medium (S3) | **Priority**: P2 | **Status**: Open
- **Expected Result**: Subsequent `DELETE` requests on an already-deleted ID should return `404 Not Found`.
- **Actual Result**: API returns `200 OK` on repeated `DELETE` calls.
