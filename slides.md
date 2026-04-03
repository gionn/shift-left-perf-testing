---
theme: default
background: https://images.unsplash.com/photo-1621537108694-3a8259512251?q=80&w=1365&auto=format&fit=crop
title: Shift Left with Performance Testing
info: |
  Performance issues are nobody's favorite surprises. When they hit, fixing them is tough and often costly. The smart move: shift left and catch problems early in the Software Development Life Cycle (SDLC). This session dives into how performance testing is built into the Alfresco delivery pipeline. We use k6 for reliable load tests, InfluxDB to save the flood of metrics, and Grafana for turning numbers into clear visuals every team can act on. The goal is simple: make performance testing a routine, automated part of your SDLC, and not a last-minute checkbox before every release.
class: text-center
drawings:
  persist: false
transition: slide-left
comark: true
duration: 40min
canvasWidth: 700
---
# Shift Left with Performance Testing

Giovanni Toraldo

Hyland

---
layout: image-right
image: https://avatars.githubusercontent.com/u/71768
---

# About me

* DevOps Engineer at Hyland
* Sometimes I get stuck in the past
* More about me on [gionn.net](https://gionn.net)

---
layout: image-right
image: https://www.svgrepo.com/show/353382/alfresco.svg
backgroundSize: contain
---

# Alfresco

Open Source document, process and governance management suite.

Alfresco is designed to help organizations efficiently manage their content and
improve productivity.

---

# Ops readiness team

My team ensures that Alfresco can be deployed and operated reliable in multiple
environments. We focus on:

* Helm charts and GitOps best practices
* Docker images and Compose for local development
* CI/CD pipelines and automation
* Performance testing and monitoring 🆕

---

# Open source projects we maintain

* Helm Charts
  * [Umbrella chart](https://github.com/Alfresco/acs-deployment) for all
    products
  * [Component charts](https://github.com/Alfresco/alfresco-helm-charts) for each
    product
* Docker images:
  [alfresco-dockerfiles-bakery](https://github.com/Alfresco/alfresco-dockerfiles-bakery)
* Ansible playbooks for classic deployments:
  [alfresco-ansible](https://github.com/Alfresco/alfresco-ansible-deployment)

---

# Table of contents

1. Performance testing basics
2. Why shift left?
3. Our solution: k6, InfluxDB, Grafana

---

# Performance

Performance is the happy problem of a software product.

It means that the software is being used, and that it is providing value to its
users.

However, when performance issues start to make the user slow, it can quickly
lead to frustration and dissatisfaction.

---
layout: two-cols-header
---

# What causes performance issues?

Performance issues can be caused by a variety of factors, including:

::left::
* Database performance
  * Missing indexes
  * Connection pool exhaustion
  * I/O bottlenecks
* High memory usage
  * Unreleased resources
  * Inefficient data structures
::right::
* CPU bottlenecks
  * Inefficient algorithms
  * Lock contention
* All of the above
  * N+1 problems
  * Network latency

---

# What is performance testing?

Performance testing validates how a system behaves under expected and
unexpected load. It answers questions like:

* How fast are key user journeys?
* How many users can we serve at once?
* What breaks first and why?

---

# What we measure

* Latency (p50, p95, p99)
* Throughput (requests per second)
* Error rates
* Resource usage (CPU, memory, I/O)

---

# Types of performance tests

* Load: expected traffic for normal conditions
* Stress: push beyond limits to find the breaking point
* Spike: sudden bursts of traffic
* Soak: long duration to find leaks and degradation

---

# Why shift left?

* Find regressions when changes are small and cheap to fix
* Keep performance as a product feature, not a release gate
* Give developers fast, actionable feedback, not customer complaints

---

# Cost of late defects

The later a performance issue is found, the more expensive it is to fix:

* **Dev**: a slow query caught in a local test costs minutes
* **CI**: caught on a PR costs an hour of review and a fix commit
* **Staging**: costs a sprint delay and cross-team coordination
* **Production**: costs user trust, on-call time, and hotfix risk

Shift left = move the discovery point as early as possible.

---

# Solution overview

We use a lightweight stack that is easy to automate:

* k6: define and run load tests
* InfluxDB: store time-series metrics
* Grafana: visualize and share results

---

# Provisioning and delivery

* Terraform pipeline provisions Alfresco on EKS
* Cluster Autoscaler ready to scale nodes during load tests
* Helm charts deploy the stack
* FluxCD keeps the latest tested tags synced
* k6 runs as a pod inside the cluster

---

# Data flow

1. k6 executes user journeys against the system under test
2. Metrics are pushed to InfluxDB (both k6 and system metrics)
3. Grafana dashboards show trends and regressions

---

# Where it runs

* Local: quick checks during development
* CI: validate changes on every merge
* Scheduled: nightly or weekly baselines

---

# Why focus on k6

* Scripted tests live with the code
* Fast, headless execution
* Built-in metrics, thresholds, and checks
* Easy to integrate with CI pipelines

---

# k6 mental model

* Virtual users (VUs) run your script in parallel
* Scenarios control arrival rate and ramping
* Iterations define how much work each VU does

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

* Tag key requests with `tags` for per-endpoint analysis
* Use consistent test names to compare runs over time

---

# k6 output formats

k6 can stream metrics to multiple backends:

```bash
# InfluxDB (v1)
k6 run --out influxdb=http://localhost:8086/k6 script.js

# JSON file for post-processing
k6 run --out json=results.json script.js

# Grafana Cloud k6
k6 run --out cloud script.js
```

We use InfluxDB v1 for self-hosted, always-on storage alongside Grafana.

---

# InfluxDB schema

k6 writes one measurement per metric type. Key measurements:

| Measurement | Tags | Fields |
| --- | --- | --- |
| `http_req_duration` | `name`, `status`, `method` | `value` |
| `http_req_failed` | `name` | `value` |
| `vus` | — | `value` |
| `iterations` | `scenario` | `value` |

Tag by `name` to get per-endpoint latency breakdowns in Grafana.

---

# Grafana dashboard

Key panels we track per test run:

* **Latency trends**: p50 / p95 / p99 over time per endpoint
* **Error rate**: percentage of failed requests
* **VU ramp**: active virtual users vs. request rate
* **Resource usage**: CPU and memory of the system under test

Dashboards are version-controlled alongside the k6 scripts.

---

# Test design tips

* Start with the top 2-3 user journeys
* Use realistic data and think time
* Keep environments consistent for baselines

---

# CI integration

Run k6 as part of your pipeline — fail the build on threshold breaches:

```yaml
- name: Run performance tests
  run: |
    k6 run \
      --out influxdb=http://influxdb:8086/k6 \
      --env BASE_URL=${{ env.APP_URL }} \
      tests/load.js
```

A non-zero exit code from k6 fails the step when any threshold is exceeded.

---

# Common pitfalls

* Unrealistic traffic patterns
* Ignoring warm-up and cache effects
* Mixing load and stress goals in one test

---

# What's next

Ideas to extend the current setup:

* Browser-level tests with k6 browser for UI journeys
* Chaos engineering: inject failures during load tests
* SLO alerting: Grafana alerts when baselines drift
* Distributed k6 runs with k6 Operator on Kubernetes

---

# Wrap-up

* Performance testing is part of the SDLC, not the release day
* k6 provides fast feedback and automation
* InfluxDB and Grafana make results visible and actionable

---
layout: image-right
image: ./images/shift-left-with-performance-testing_toraldo_1129276_feedback-code.png
---

# Questions?

Reach me on LinkedIn: [Giovanni Toraldo](https://www.linkedin.com/in/giovannitoraldo/)

or on my website [gionn.net](https://gionn.net)

Slides sources on GitHub: [github.com/gionn/shift-left-performance-testing](https://github.com/gionn/shift-left-performance-testing)
