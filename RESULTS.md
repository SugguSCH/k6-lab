# K6 lab results

## Part 0

- Step 0.1, `http_req_duration`: 286.82 ms (one request; avg, min, med, max, p(90), and p(95) are equal). The status 200 check passed (1/1), and `http_req_failed` was 0%.
- Step 0.2, 5 VUs for 20 seconds: 80 HTTP requests, 3.787753 requests/second, and `http_req_duration` p(95) of 331.69 ms. All 80 checks passed, with 0% failed requests. The request rate stayed below 5/second because each user waits for the HTTP response and then sleeps for 1 second before starting the next iteration; one user cannot make one request every second.
- Step 0.3, 5 VUs sharing 20 iterations: exactly 20 iterations and 20 HTTP requests completed in 5.7 seconds. All 20 checks passed, with 0% failed requests. Unlike Step 0.2, this run stopped when the fixed amount of work finished.
- Step 0.4, vegetarian pizza API: all three checks passed (status 200, a pizza name, and only vegetarian ingredients). `http_req_duration` was 304.61 ms, and `http_req_failed` was 0%.
- Step 0.5, request without a token: QuickPizza returned the expected 401 status. The check passed (1/1), and `http_req_failed` stayed at 0% because the script marks 401 as an expected response.
- Step 0.6, default `P95=1500`: all three thresholds passed; measured p(95) was 388.58 ms, checks were 100%, failed requests were 0%, and the exit code was 0. With `P95=300`, the response-time threshold failed because p(95) was 408.31 ms; checks were still 100% and failed requests 0%, but the exit code was 99.
