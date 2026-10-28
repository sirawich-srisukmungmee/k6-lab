
# K6 Lab Results

## Part 0: K6 Basics

### Step 0.1 — First Request
- HTTP status: 200
- Checks passed: 1/1 (100%)
- Total requests: 1
- Average response time: 369.72 ms
- p(95): 369.72 ms

### Step 0.2 — Load Test with 5 VUs
- Virtual users: 5
- Duration: 20 seconds
- Total requests: 75
- Requests per second: 3.682249
- Average response time: 311.46 ms
- Median response time: 322.02 ms
- p(95): 358.99 ms
- Maximum response time: 373.05 ms
- Failed requests: 0%
- Checks passed: 75/75 (100%)

**Why was the request rate below 5 requests per second?**

Each iteration includes a 1-second sleep, and each HTTP request also takes time to complete. Therefore, each virtual user cannot complete five requests every second.

### Step 0.3 — Fixed Iterations
- Total iterations: 20
- Total requests: 20
- Requests per second: 3.310354
- Average response time: 337.63 ms
- p(95): 363.35 ms
- Failed requests: 0%
- Checks passed: 20/20 (100%)

### Step 0.4 — Vegetarian Pizza API
- HTTP status: 200
- Checks passed: 3/3 (100%)
- Average response time: 338.4 ms
- Failed requests: 0%

### Step 0.5 — Request Without a Token
- The no-token check passed.
- The request was refused as expected with HTTP 401.
- Checks passed: 1/1 (100%)
- `http_req_failed`: 0%

An expected HTTP 401 response can still count as a successful HTTP request in k6.

### Step 0.6 — Thresholds
- Normal run: all configured thresholds passed.
- Normal run p(95): 378.16 ms.
- Normal run failed-request rate: 0%.
- Run with `P95=300`: the p(95) threshold failed because measured p(95) was 374.97 ms.
- Exact process exit codes: not recorded.

### Step 0.7 — Measurable Performance Requirement
- Normal run p(95): 376.03 ms.
- Normal run p(99): 380.29 ms.
- Normal run thresholds: passed.
- Failed-request rate: 0%.
- After changing the requirement to `p(95)<1`, the p(95) threshold failed.
- Measured p(95) after changing the requirement: 375.64 ms.
- Exact process exit code: not recorded.

### Step 0.8 — Bug Hunt
- Initial run:
  - Total requests: 75
  - Checks passed: 189/225 (84%)
  - Checks failed: 36/225 (16%)
- Investigation:
  - The toppings API returned the name `Pepperoni`.
  - The tools API listed `Knife` among the available tools.
- Fix tested:
  - `excludedIngredients: ['Pepperoni']`
  - `excludedTools: ['Knife']`
- Final run:
  - Total requests: 75
  - Requests per second: 3.557479
  - Average response time: 341.48 ms
  - p(95): 377.55 ms
  - Maximum response time: 475.79 ms
  - Failed requests: 0%
  - Checks passed: 225/225 (100%)

**Why did the checks find the problem?**

The test repeated requests and checked every returned pizza. This helped reveal incorrect recommendations across multiple iterations that might not appear in a single manual request.

## Part A: Load Test Results

| Step | Test Type | VUs | Duration | Requests | Req/s | Average Response Time | p(95) | Max | Failed Requests | Checks |
|---|---|---:|---|---:|---:|---:|---:|---:|---:|---:|
| A1 | Smoke | 1 | 30 s | 15 | 0.473523 | 316.08 ms | 377.72 ms | 378.58 ms | 0% | 20/20 (100%) |
| A2 | Average Load | 3 max | 3 min | 234 | 1.271962 | 315.36 ms | 378.70 ms | 475.22 ms | 0% | 312/312 (100%) |
| A3 | Stress | 5 max | 3 min | 384 | 2.127304 | 313.61 ms | 377.81 ms | 457.88 ms | 0% | 512/512 (100%) |
| A4 | Spike | 5 max | 2 min 20 s | 147 | 1.042811 | 328.28 ms | 378.22 ms | 1.19 s | 0% | 196/196 (100%) |
| A5 | Soak | 2 max | 11 min | 636 | 0.962398 | 327.46 ms | 387.81 ms | 946.66 ms | 0% | 848/848 (100%) |

### Part A Summary
- All tests from A1 to A5 passed their configured checks.
- All five tests reported 0% failed HTTP requests.
- A2 p(95): 378.70 ms.
- A3 p(95): 377.81 ms.
- A4 p(95): 378.22 ms; maximum response time: 1.19 seconds.
- A5 p(95): 387.81 ms; maximum response time: 946.66 ms.
- The terminal summaries do not show the exact spike-recovery duration or prove whether response time steadily increased throughout the soak test.

## Part B: Questions

### 1. Summarize the results of Steps A1–A5.
All five tests passed their configured checks and reported 0% failed HTTP requests. A2 and A3 had similar aggregate p(95) response times of 378.70 ms and 377.81 ms. A4 reached a maximum response time of 1.19 seconds. A5 recorded a p(95) of 387.81 ms and a maximum of 946.66 ms.

### 2. Why was the request rate below 5 requests per second in Step 0.2?
Each iteration includes a 1-second sleep, and the HTTP request itself takes additional time. Therefore, each virtual user completes fewer than five requests per second.

### 3. Did response time grow from the average-load test to the stress test? Explain.
No increase was observed in aggregate p(95). A2 recorded 378.70 ms, while A3 recorded 377.81 ms. Both tests reported 0% failed HTTP requests. Under these specific test conditions, response time remained stable as the load increased.

### 4. How long did response time take to recover after the spike?
The A4 terminal summary shows a maximum response time of 1.19 seconds and an aggregate p(95) of 378.22 ms, but it does not show the exact recovery duration. The recovery time must be determined from the response-time chart in `reports/a4-spike.html`.

### 5. Did response time or error rate get worse over time in the soak test? What defect might a longer soak test reveal?
A5 recorded p(95) of 387.81 ms, a maximum response time of 946.66 ms, and 0% failed HTTP requests. The aggregate results alone cannot confirm whether response time increased over time. A longer soak test might reveal a memory leak, resource exhaustion, connection-pool exhaustion, or gradual performance degradation.

### 6. Why use p(95) instead of average response time?
The average can hide unusually slow requests. p(95) shows the response-time value within which 95% of requests completed, giving a better view of slower user experiences.

### 7. Why might the same tests produce different results in a lab environment?
Results can vary because of network conditions, shared server load, other users' traffic, local computer resources, and timing differences.

### 8. Why must you not run a test with hundreds of users against a public site without permission?
A large test could overload or disrupt a public service, affect real users, and violate usage policies. The lab limits must be followed: no more than 5 VUs for the short load tests and 2 VUs for the soak test.

### 9. Give a vague performance requirement and rewrite it as a measurable requirement with k6 thresholds.

**Vague requirement:**  
The application should be fast and reliable.

**Measurable requirement:**  
Under the defined test workload, at least 99% of checks must pass, fewer than 1% of HTTP requests may fail, the home-page p(95) response time must be below 1,000 ms, and the recommendation endpoint's p(95) response time must be below 1,500 ms.

Example k6 thresholds:

```javascript
thresholds: {
  checks: ['rate>0.99'],
  http_req_failed: ['rate<0.01'],
  'http_req_duration{group:::home}': ['p(95)<1000'],
  'http_req_duration{group:::recommendation}': ['p(95)<1500'],
}
```

## HTML Reports
- A2: `reports/a2-average.html`
- A3: `reports/a3-stress.html`
- A4: `reports/a4-spike.html`
- A5: `reports/a5-soak.html`

## Notes Before Submission
- Exact exit codes for the threshold-failure runs in Steps 0.6 and 0.7 were not recorded.
- Review `reports/a4-spike.html` to determine the exact recovery duration for Part B Question 4.
- Review `reports/a5-soak.html` to determine whether response time or error rate changed over time for Part B Question 5.
