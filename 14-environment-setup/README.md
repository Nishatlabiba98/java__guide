# 14 — Environment Setup

How TapRoot's environments differ and how to work with each one.

- **Local/dev** — running the app on your own machine, local config, hitting local or shared dev dependencies
- **SBX (sandbox)** — shared testing environment, how it differs from local, who else uses it
- **UAT** — user acceptance testing, what gates a change getting promoted here
- **PROD** — production, access restrictions, deploy process, why nothing skips UAT

Ties directly into [16 — Spring](../16-spring/README.md)'s config/profiles section — each environment above typically maps to a Spring profile (`application-dev.properties`, `application-uat.properties`, etc.) that swaps things like DB connection strings and feature flags without changing code.

## Shell commands you'll actually use

Regardless of environment, a baseline of comfort with the shell (from [00 — Setup](../00-setup/README.md)) pays off here — checking a running process, tailing a log, or inspecting disk space are all things you'll do against dev/SBX/UAT boxes routinely:

```
ps aux | grep java        # find a running Java process
tail -f app.log           # follow a log file live
grep ERROR app.log        # search a log for a pattern
df -h                     # disk space
curl -I https://host/health  # quick health-check hit
```

## Monitoring

Once code is running somewhere other than your laptop, you need visibility into whether it's actually working:

- **Logs** — the first thing you check when something's wrong. Structured logging (consistent fields like timestamp, level, request ID) makes searching across a fleet of instances possible; ad-hoc `System.out.println` doesn't.
- **Metrics** — numeric time-series data (request rate, error rate, response time, memory usage) aggregated on a dashboard, so you can see trends and set alerts instead of reading logs line by line.
- **Health checks** — a simple endpoint (`/health` or `/actuator/health` in Spring Boot) that reports whether the app is up and its dependencies (database, downstream services) are reachable — what load balancers and deploy tooling poll to decide if an instance is safe to route traffic to.

Which specific tools TapRoot uses (log aggregator, metrics dashboard) is worth confirming with your team — the concepts above are what to look for regardless of the specific product.

## Checklist

- [ ] Can explain what distinguishes local/dev, SBX, UAT, and PROD
- [ ] Comfortable checking a running process and tailing/searching a log file from the shell
- [ ] Can explain the difference between logs, metrics, and health checks
- [ ] Know that environment-specific config in a Spring app maps to profiles (section 16)
