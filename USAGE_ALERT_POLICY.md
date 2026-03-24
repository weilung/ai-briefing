# Usage Alert Policy (Cron v1)

## Goal
Detect real usage risk while suppressing false alarms from temporary missing metrics.

## Schedule
- Check interval: every **30 minutes**
- Quiet hours (optional): 23:00–08:00 only send **critical** alerts

## Data Source
- Use `session_status` as primary source
- Parse fields: daily left %, weekly left %, tokens in/out (if available)

## Thresholds
- **Warning**: daily remaining <= 35%
- **High**: daily remaining <= 20%
- **Critical**: daily remaining <= 10%
- **Weekly warning**: weekly remaining <= 25%

## Retry & Debounce
When check fails (no usage field / timeout / parse error):
1. Mark status = transient_error
2. Retry after **5 minutes**
3. Retry after **10 minutes** (2nd retry)
4. Only alert "monitoring issue" if **3 consecutive failures**

When usage is over threshold:
- Require **2 consecutive checks** to confirm before sending warning/high
- Critical can alert immediately on first valid detection

## Notification Rules
- Send at most one alert per level per 6 hours (cooldown)
- Escalate immediately if level increases (Warning -> High -> Critical)
- Include:
  - current daily/weekly remaining
  - previous check value
  - recommended action

## Suggested Alert Message Template
- Warning:
  - "Usage warning: daily remaining {x}%, weekly remaining {y}%. Consider lowering Think level / delaying heavy jobs."
- High:
  - "Usage high risk: daily remaining {x}%. Recommend pausing nonessential background runs."
- Critical:
  - "Usage critical: daily remaining {x}%. Immediate action recommended (stop heavy tasks, switch to low Think)."
- Monitoring issue:
  - "Usage monitor issue: 3 consecutive check failures. Will continue retrying every 30 min."

## Operational Notes
- Missing usage once is normal; do not notify user immediately
- Cache-heavy periods may show low token movement; use remaining % as primary signal
- Keep state in a small json file, e.g. `ai-briefing/state/usage-monitor.json`
