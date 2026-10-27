# Banking load tests with JMeter

[![tests](https://github.com/gamzesimit/banking-jmeter-load/actions/workflows/tests.yml/badge.svg)](https://github.com/gamzesimit/banking-jmeter-load/actions/workflows/tests.yml)

An Apache JMeter plan for a retail banking API, run headless and on every commit.

Results: [docs-results.md](docs-results.md)

## Running it

```bash
docker compose up -d
docker run --rm --network host -v "$PWD":/work -w /work justb4/jmeter:5.5 \
  -n -t plans/read-endpoints.jmx -l results/run.jtl \
  -Jhost=localhost -Jport=8081 -Jusers=10 -Jduration=20
```

Thread count, duration, host and port are all plan properties, so the same file
runs against a laptop and against a deployed environment without being edited.

## Why the assertions are inside the plan

A load run that only counts errors will call a 200 carrying an empty body a
success. Every sample here checks the status code, checks the body carries a
balance, and checks the response arrived inside 1500 ms. A fast wrong answer
fails.

## What this is not

It is not a capacity measurement. The client and the server sit on the same
machine, which removes the network and flatters every figure. What it is good
for is noticing the day the floor moves, which is why it runs on every commit
rather than once before a release.

## Relationship to the API tests

The same endpoints are checked for correctness in
[banking-api-tests](https://github.com/gamzesimit/banking-api-tests), where every
endpoint also carries a single-request response time budget. That suite answers
"is each answer right and fast enough on its own"; this plan answers "does it stay
right and fast under sustained load".
