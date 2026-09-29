# Load test results

Plan: `plans/read-endpoints.jmx`
Tool: Apache JMeter 5.5, run headless
Profile: 10 threads, 5 second ramp, 20 second hold, read only
Target: the account and customer account endpoints on a local container

## Assertions inside the plan

Every sample carries three checks, so a fast response that is wrong still fails:

- The response code is 200
- The body carries a balance
- The response arrives inside 1500 ms

## Result

| Measure | Value |
|---|---|
| Samples | 23,959 |
| Throughput | 1,176 per second |
| Average | 6 ms |
| Minimum | 0 ms |
| Maximum | 239 ms |
| Errors | 0.00 per cent |

## How to read this

The client and the server are on the same machine, so there is no network
between them. These numbers say the application does not fall over under a
steady read load and that the response shape stays correct while it is busy.
They do not predict anything about a deployed environment.

The number worth keeping is the gap between the average at 6 ms and the maximum
at 239 ms. That shape usually means a cold cache or a garbage collection pause
rather than a steady slowdown, and it is the first thing to re-measure if a
threshold starts failing.

## Reproducing

```bash
docker compose up -d
docker run --rm --network host -v "$PWD":/work -w /work justb4/jmeter:5.5 \
  -n -t plans/read-endpoints.jmx -l results/run.jtl \
  -Jhost=localhost -Jport=8081 -Jusers=10 -Jduration=20
```

An HTML report from the same results file:

```bash
docker run --rm --network host -v "$PWD":/work -w /work justb4/jmeter:5.5 \
  -g results/run.jtl -o results/html
```
