---
theme: default
background: https://cover.sli.dev
title: Shift Left with Performance Testing
info: |
  Performance issues are nobody's favorite surprises. When they hit, fixing them is tough and often costly. The smart move: shift left and catch problems early in the Software Development Life Cycle (SDLC). This session dives into how performance testing is built into the Alfresco delivery pipeline. We use k6 for reliable load tests, InfluxDB to save the flood of metrics, and Grafana for turning numbers into clear visuals every team can act on. The goal is simple: make performance testing a routine, automated part of your SDLC, and not a last-minute checkbox before every release.
class: text-center
drawings:
  persist: false
transition: slide-left
comark: true
duration: 40min
---
# Shift Left with Performance Testing

Giovanni Toraldo

Hyland

---

# About me

- 👋Hi, my name is Giovanni Toraldo and I work in Hyland.

---

# Table of contents

<Toc minDepth="1" maxDepth="1" />


---

# Alfresco

Alfresco is an open-source content management system (CMS) that provides a
platform for managing and sharing digital content.

It offers features such as document management, collaboration, workflow
automation, and records management.

Alfresco is designed to help organizations efficiently manage their content and
improve productivity.

---

# Performance

Performance is the happy problem of a software product.

It means that the software is being used, and that it is providing value to its
users.

However, when performance issues start to make the user slow, it can quickly
lead to frustration and dissatisfaction.

---

# What causes performance issues?

Performance issues can be caused by a variety of factors, including:

- Database performance
  - Missing indexes
  - Connection pool exhaustion
  - I/O bottlenecks
- High memory usage
  - Unreleased resources
  - Inefficient data structures
- CPU bottlenecks
  - Inefficient algorithms
  - Lock contention
- All of the above
  - N+1 problems
  - Network latency

---

# What is performance testing?

Performance testing validates how a system behaves under expected and
unexpected load. It answers questions like:

- How fast are key user journeys?
- How many users can we serve at once?
- What breaks first and why?

---

# What we measure

- Latency (p50, p95, p99)
- Throughput (requests per second)
- Error rates
- Resource usage (CPU, memory, I/O)

---

# Types of performance tests

- Load: expected traffic for normal conditions
- Stress: push beyond limits to find the breaking point
- Spike: sudden bursts of traffic
- Soak: long duration to find leaks and degradation

---

# Why shift left?

- Find regressions when changes are small and cheap to fix
- Keep performance as a product feature, not a release gate
- Give developers fast, actionable feedback

---

# Solution overview

We use a lightweight stack that is easy to automate:

- k6: define and run load tests
- InfluxDB: store time-series metrics
- Grafana: visualize and share results

---

# Provisioning and delivery

- Terraform pipeline provisions Alfresco on EKS
- Cluster Autoscaler ready to scale nodes during load tests
- Helm charts deploy the stack
- FluxCD keeps the latest tested tags synced
- k6 runs as a pod inside the cluster

---

# Data flow

1. k6 executes user journeys against the system under test
2. Metrics are pushed to InfluxDB
3. Grafana dashboards show trends and regressions

---

# Where it runs

- Local: quick checks during development
- CI: validate changes on every merge
- Scheduled: nightly or weekly baselines

---

# Why focus on k6

- Scripted tests live with the code
- Fast, headless execution
- Built-in metrics, thresholds, and checks
- Easy to integrate with CI pipelines

---

# k6 mental model

- Virtual users (VUs) run your script in parallel
- Scenarios control arrival rate and ramping
- Iterations define how much work each VU does

---

# k6 test structure

```js
import http from "k6/http"
import { check, sleep } from "k6"

export const options = {
  vus: 10,
  duration: "30s",
}

export default function () {
  const res = http.get("https://example.com")
  check(res, {
    "status is 200": (r) => r.status === 200,
  })
  sleep(1)
}
```

---

# Scenarios and ramping

```js
export const options = {
  scenarios: {
    steady: {
      executor: "ramping-vus",
      stages: [
        { duration: "30s", target: 10 },
        { duration: "2m", target: 10 },
        { duration: "30s", target: 0 },
      ],
    },
  },
}
```

---

# Thresholds and checks

```js
export const options = {
  thresholds: {
    http_req_failed: ["rate<0.01"],
    http_req_duration: ["p(95)<500"],
  },
}
```

---

# Tags and trends

- Tag key requests with `tags` for per-endpoint analysis
- Use consistent test names to compare runs over time

---

# Test design tips

- Start with the top 2-3 user journeys
- Use realistic data and think time
- Keep environments consistent for baselines

---

# Common pitfalls

- Unrealistic traffic patterns
- Ignoring warm-up and cache effects
- Mixing load and stress goals in one test

---

# Wrap-up

- Performance testing is part of the SDLC, not the release day
- k6 provides fast feedback and automation
- InfluxDB and Grafana make results visible and actionable
  