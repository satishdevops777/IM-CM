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



# 🚨 Real Production Incident Scenarios – SRE Runbook (With Commands & Alerts)

This document contains **real production incidents** encountered in Linux, Kubernetes, and cloud environments.

Each scenario includes:
- What broke
- Alerts that fired
- Commands used during investigation
- Resolution steps
- Preventive actions

---

## 🔥 Scenario 1: Bad Deployment → Application CrashLoop

### Incident
New application version deployed to production caused service outage.

### Alerts
- Kubernetes pod restarts > threshold
- HTTP 5xx error rate spike
- Service availability < SLO

### Investigation Commands
```bash
kubectl get pods
kubectl describe pod <pod-name>
kubectl logs <pod-name>
kubectl get events --sort-by=.metadata.creationTimestamp
```

### Findings
- Pods in `CrashLoopBackOff`
- `OOMKilled` in pod events

### Resolution
```bash
kubectl rollout undo deployment app-deployment
```

### Root Cause
- Memory leak in new release
- No load testing

### Prevention
- Canary deployments
- Load tests in CI
- Resource limits enforced

---

## 🔥 Scenario 2: Database Connection Pool Exhaustion

### Incident
Application latency increased drastically.

### Alerts
- API latency > threshold
- DB connections > 90%
- Error rate spike

### Investigation Commands
```bash
top
netstat -an | grep 5432 | wc -l
lsof -i :5432
```

Database:
```sql
SELECT count(*) FROM pg_stat_activity;
```

### Findings
- DB hit max connections
- Requests blocked

### Resolution
```bash
systemctl restart application
```

### Root Cause
- Misconfigured connection pool
- No saturation alerts

### Prevention
- Connection pooling
- DB alerts
- Load testing

---

## 🔥 Scenario 3: Disk Full → Application Crash

### Incident
Application stopped responding.

### Alerts
- Disk usage > 90%
- Node filesystem alerts

### Investigation Commands
```bash
df -h
du -sh /var/log/*
ls -lh /var/log
```

### Findings
- Logs filled entire disk
- Log rotation broken

### Resolution
```bash
rm -f /var/log/*.log
systemctl restart app
```

### Root Cause
- Logrotate misconfigured
- No disk alerts

### Prevention
- Disk alerts at 70/80/90%
- Logrotate validation

---

## 🔥 Scenario 4: Kubernetes Node Failure

### Incident
Worker node went down.

### Alerts
- Node NotReady
- Pod availability below threshold

### Investigation Commands
```bash
kubectl get nodes
kubectl describe node <node-name>
kubectl get pods -o wide
```

### Findings
- Node ran out of memory
- Pods had no limits

### Resolution
- Pods rescheduled automatically
- Node replaced

### Root Cause
- Missing resource limits
- No cluster autoscaling

### Prevention
- Enforce limits
- Cluster autoscaler

---

## 🔥 Scenario 5: DNS Misconfiguration

### Incident
Service unreachable via domain name.

### Alerts
- Synthetic check failures
- Traffic dropped to zero

### Investigation Commands
```bash
nslookup example.com
dig example.com
curl -v https://example.com
```

### Findings
- DNS record pointed to wrong IP
- High TTL delayed recovery

### Resolution
- Fixed DNS record
- Waited for TTL expiry

### Root Cause
- No DNS validation
- Manual DNS change

### Prevention
- DNS change review
- Lower TTLs

---

## 🔥 Scenario 6: Certificate Expiry

### Incident
HTTPS stopped working.

### Alerts
- SSL certificate expiry warning (if configured)

### Investigation Commands
```bash
openssl s_client -connect example.com:443
```

### Findings
- Certificate expired
- No renewal automation

### Resolution
```bash
certbot renew
systemctl reload nginx
```

### Prevention
- Certificate expiry monitoring
- Automated renewals

---

## 🔥 Scenario 7: Traffic Spike Overwhelmed Service

### Incident
Sudden spike caused timeouts.

### Alerts
- CPU > 90%
- Latency SLO breach

### Investigation Commands
```bash
kubectl get hpa
kubectl top pods
kubectl top nodes
```

### Resolution
```bash
kubectl scale deployment app --replicas=10
```

### Prevention
- Enable HPA
- Load testing

---

## 🔥 Scenario 8: Monitoring Failure Hid Outage

### Incident
Service down but no alerts fired.

### Investigation Commands
```bash
systemctl status monitoring-agent
ps -ef | grep agent
```

### Findings
- Monitoring agent crashed

### Resolution
```bash
systemctl restart monitoring-agent
```

### Prevention
- Alert on missing metrics
- Redundant monitoring

---

## 🧠 SRE Golden Rule

Incidents are rarely caused by a single mistake — they are caused by **system gaps**.

