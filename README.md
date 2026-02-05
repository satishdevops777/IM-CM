# IM-CM

- Incident Management is the process of restoring service as quickly as possible when something breaks in production.

🎯 Goal: Reduce downtime & user impact
❌ Not about blame
❌ Not about permanent fixes (that’s Problem Management)

### What counts as an incident?
- Website down ❌
- API latency suddenly spikes ⏱️
- Database not accepting connections 🛑
- Kubernetes pods in CrashLoopBackOff 🔄
- CPU / memory exhausted on prod servers 💥

### Incident Management Lifecycle (Simple)
### 1️⃣ Detection
- Alert from monitoring (Prometheus, Splunk, SignalFx)
- User complaints
- Synthetic checks failing

### 2️⃣ Triage
- Is it real?
- How severe is it?
- Who is impacted?

### 3️⃣ Response & Mitigation
- Roll back
- Restart service
- Scale resources
- Disable a feature flag

### 4️⃣ Resolution
- Service fully restored

### 5️⃣ Post-Incident Review (PIR / RCA)
- What happened?
- Why?
- How to prevent next time?

### Severity Levels (Typical)
| Severity | Meaning           | Example             |
| -------- | ----------------- | ------------------- |
| **P1**   | Critical outage   | Payment system down |
| **P2**   | Major degradation | 50% API errors      |
| **P3**   | Minor issue       | One pod failing     |
| **P4**   | Low impact        | Log rotation issue  |

- Change Management controls how changes are introduced into production to avoid incidents.

### What is a “change”?
- Deploying new code 🚀
- Updating OS packages 🧱
- Scaling infrastructure 📈
- Modifying firewall rules 🔐
- Database schema changes 🗄️

### Types of Changes
| Type          | Description          | Example                |
| ------------- | -------------------- | ---------------------- |
| **Standard**  | Low risk, repeatable | Restarting a service   |
| **Normal**    | Needs approval       | New feature release    |
| **Emergency** | Fix prod outage      | Hotfix during incident |

### Change Management Flow (Modern / DevOps)

### 1️⃣ Plan
- What is changing?
- Risk level?
- Rollback plan?

### 2️⃣ Review / Approval
- Peer review
- CAB (in traditional orgs)

### 3️⃣ Implement
- CI/CD pipeline
- Infra as Code (Terraform)

### 4️⃣ Validate
- Smoke tests
- Monitoring

### 5️⃣ Close
- Document results


### Real Example – Change

- Scenario:
Deploying a new API version.
- Good change management:
- Canary deployment to 5% traffic
- Monitor error rate
- Gradually increase traffic
- Full rollout only after stability confirmed
Bad change management:
- Friday evening deployment
- No rollback plan
- No monitoring
- Causes P1 incident 😬

| Aspect  | Incident Management     | Change Management |
| ------- | ----------------------- | ----------------- |
| Trigger | Something breaks        | Planned activity  |
| Goal    | Restore service fast    | Prevent outages   |
| Timing  | Reactive                | Proactive         |
| Risk    | Already impacting users | Controlled risk   |
| Example | DB outage               | App deployment    |

Why?

How to prevent next time?
