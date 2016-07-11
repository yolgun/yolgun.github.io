---
title: Site Reliability Engineering - Notes
layout: post
author: yunolgun
permalink: /site-reliability-engineering---notes/
source-id: 1pxEcqinpJ7afV4pKyH5gD4vrS29js7J0OQh9-lRN3co
published: true
---
1. **Introduction**

* Hope is not a strategy.

* Traditional sysadmins approach is not scalable with traffic.

* Sysadmins want stability; developers want features. This may cause strife between teams.

* UNIX internals and networking(Layer 1 to Layer 3) knowledge is a plus for SRE work.

* 50% cap on "ops" work vs development for SREs.

* Availability, Latency, Performance, Efficiency, Change, Monitoring, Emergency, Capacity.

* 100% reliability target is hard to achieve and almost always unnecessary.

* Remaining time from SLO(e.g. 99.9% availability) makes **error budget**. Spend it on new features.

* Software should monitor and humans should only be alerted when they need to take action.

* **Monitoring** output: Alerts(immediate action), tickets(relaxed action), logging(only when asked to look).

* Disaster playbooks are very helpful to reduce MTTR(mean time to repair) and improve **emergency response**.

* **Change**: Progressive rollouts -> Detect problems -> Roll back in case of problems.

* **Capacity Planning**: Organic and inorganic demand casting, regular load testing.

