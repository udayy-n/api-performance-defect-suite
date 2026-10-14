# API Performance Testing & Defect Management Suite

Automated functional/regression API test collections, CLI test execution, 100-user load testing, and structured Jira defect reporting using **Postman**, **Newman CLI**, **Apache JMeter**, and **Jira**.

## Key Features Matching Project Scope
- **Postman Functional & Regression Collection (`postman/`)**: 8 automated API requests with **17 JavaScript assertions (`pm.test`)** validating status codes (`200`, `201`, `404`), SLA response times (`< 2000ms`), response headers, and JSON payloads across `GET`, `POST`, `PUT`, `PATCH`, and `DELETE`.
- **Newman CLI & GitHub Actions Automation (`.github/workflows/newman-ci.yml`)**: Command-line execution of Postman collections with JUnit/CLI report generation.
- **Apache JMeter Load Testing (`jmeter/API_Load_Test_100_Concurrent_Users.jmx`)**: Simulates **100 concurrent virtual users** over a 30-second ramp-up period to measure average response time, 90th/95th percentile latency, throughput (`req/sec`), and error rates.
- **Structured Jira Defect Logs (`defect-reports/`)**: Detailed defect reports (`BUG-101` – `BUG-105`) in both Markdown and CSV (Jira importable) formats with severity, priority, reproduction steps, and root-cause analysis.

## How to Run Locally
### 1. Run Postman Collection via Newman CLI (or Import into Postman Desktop)
```bash
npx newman run postman/REST_API_Functional_Regression.postman_collection.json -e postman/QA_Environment.postman_environment.json
```

### 2. Run or Inspect Apache JMeter Load Test Plan
Open `jmeter/API_Load_Test_100_Concurrent_Users.jmx` in Apache JMeter GUI, or run in non-GUI CLI mode:
```bash
jmeter -n -t jmeter/API_Load_Test_100_Concurrent_Users.jmx -l results.jtl
```
