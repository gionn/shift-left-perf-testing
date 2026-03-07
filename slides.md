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
  