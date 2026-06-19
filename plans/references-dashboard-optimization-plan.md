---
source_path: /root/.hermes/skills/hermes-dashboard/references/dashboard-optimization-plan.md
imported_at: 2026-06-19T13:19:46
category: plans
---

# Dashboard Optimization Plan Notes

Session-derived implementation notes for applying `hermes-optimization-plan.md`-style upgrades to the Hermes management dashboard and system monitor.

## Management Dashboard (`/opt/hermes-dashboard`, port 9100)

### Global time range

- Add a single frontend state such as `let currentDays = 7`.
- Add a `.time-range-bar` near the top of `<main>` with buttons for `24h`, `7天`, `30天`, `90天`.
- On click, update `currentDays`, toggle the active button, and call `refreshCurrentTab()`.
- Thread `currentDays` through existing endpoint calls:
  - `/api/token-trend?days=${currentDays}`
  - `/api/models-usage?days=${currentDays}`
  - `/api/platforms-usage?days=${currentDays}`
  - `/api/cache-hit-rate?days=${currentDays}`

### Cost trend

Backend pattern:

```python
@app.get("/api/cost-trend")
async def cost_trend(days: int = Query(7, ge=1, le=365)):
    rows = fetch_all(f"""
        SELECT date(datetime(started_at, 'unixepoch')) AS day,
               COALESCE(SUM(estimated_cost_usd), 0) AS cost,
               COALESCE(SUM(input_tokens), 0) AS input_tokens,
               COALESCE(SUM(output_tokens), 0) AS output_tokens,
               COUNT(*) AS sessions
        FROM sessions
        WHERE started_at >= strftime('%s', 'now', '-{days} days')
        GROUP BY day
        ORDER BY day ASC
    """)
    return {"dates": [...], "cost": [...], ...}
```

Frontend pattern:

- Add `chartCostTrend` card to Overview.
- Fetch `/api/cost-trend?days=${currentDays}` alongside token/model/platform calls.
- Render with ECharts line chart using dollars in tooltip and y-axis.

## System Monitor (`/opt/hermes-system-monitor`, port 9000)

### Disk IO collection

Collector pattern:

```python
def _get_disk_io_rates(prev, interval):
    current = psutil.disk_io_counters()
    # delta read/write bytes ÷ interval ÷ 1024**2 => MB/s
```

- Initialize disk IO rates as `0.0` in `collect()` under `disk`.
- Keep `prev_disk` in `_run()` just like `prev_net`.
- Store `read_rate_mbps` / `write_rate_mbps` on `data["disk"]`.

Database pattern:

- Extend `CREATE TABLE` with `disk_read_rate_mbps` and `disk_write_rate_mbps`.
- In `initialize()`, run guarded migrations:

```python
for column in ("disk_read_rate_mbps", "disk_write_rate_mbps"):
    try:
        conn.execute(f"ALTER TABLE metrics ADD COLUMN {column} REAL")
    except sqlite3.OperationalError:
        pass
```

- Add the two columns to INSERT params, `_row_to_dict`, raw history arrays, and aggregated history SQL.

### Frontend alerts

- Add `.alert-banner` near the top of `.main`.
- Track `lastAlertKey` and `alertCooldownUntil` to avoid toast spam.
- Alert thresholds used in session:
  - CPU > 90%
  - memory > 90%
  - disk > 85%
- Alert effects: inline banner, temporary toast, and `document.title = '🔴 ...'`. Restore the original title when clear.

### Disk IO chart

- Add `ch-disk-io` to the chart grid and `initCharts()` list.
- Include `disk_read` and `disk_write` arrays in history responses.
- Render read/write as MB/s line series.

### Top processes

- If the frontend has process list CSS and `updateTopProcs()` already exists, restore the hidden container:

```html
<div class="proc-wrap">
  <div class="ph-head"><span>🔥</span>Top 进程</div>
  <div id="proc-list"><div class="proc-empty">加载中...</div></div>
</div>
```

- In `refreshDetailed()`, call `updateTopProcs(d.top_processes)`.

## Verification

Use code-level checks first:

```bash
python3 -m py_compile /opt/hermes-dashboard/backend.py
python3 -m py_compile /opt/hermes-system-monitor/collector.py /opt/hermes-system-monitor/database.py /opt/hermes-system-monitor/main.py
node --check /tmp/dashboard-inline.js
node --check /tmp/monitor-inline.js
```

For database migration, test fresh DB plus collection:

```python
from collector import collector
from database import MetricsDatabase
m = collector.collect()
db = MetricsDatabase('/tmp/hermes-monitor-test.db')
db.initialize()
db.insert(m)
assert 'read_rate_mbps' in db.get_latest()['disk']
assert 'disk_read' in db.get_history('1h')
```
