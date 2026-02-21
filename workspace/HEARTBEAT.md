# EagleEye — Heartbeat

## System Monitoring

### Disk Usage Check
```
Frequency: Every 30 minutes during active recon
Threshold: Alert at 75% usage
Reason: Large scan outputs can fill disk quickly
```

### Alert Protocol
```
IF disk > 75%:
  → Telegram: "DISK WARNING | Usage: X% | Action needed"
  → Pause non-critical scans
  → Await operator guidance

IF disk > 90%:
  → Telegram: "DISK CRITICAL | Usage: X% | Stopping scans"
  → Stop all active scans
  → Preserve existing outputs
  → Await operator action
```

---

## Session Health

### Context Window Monitoring
```
Frequency: Every phase completion
Threshold: Warning at 150K tokens (~75%)

IF context > 150K:
  → Summarize prior phases
  → Reference files by path
  → Clear intermediate data from context
```

### Session Time Monitoring
```
Frequency: Every phase completion
Threshold: Warning at 4 hours (of 5-hour limit)

IF elapsed > 4 hours:
  → Notify operator of remaining time
  → Prioritize critical phases
  → Plan checkpoint for session handoff
```

---

## Automated Checks

### Pre-Scan Health Check
```bash
# Run before starting recon
echo "=== Pre-Scan Health Check ==="
echo "Disk: $(df -h / | tail -1 | awk '{print $5}')"
echo "Memory: $(free -h | grep Mem | awk '{print $3"/"$2}')"
echo "Tools: $(which subfinder httpx nuclei nmap 2>/dev/null | wc -l)/4 available"
echo "Network: $(ping -c 1 8.8.8.8 2>/dev/null && echo 'OK' || echo 'FAIL')"
```

### Post-Phase Health Check
```bash
# Run after each phase
DISK=$(df / | tail -1 | awk '{print $5}' | tr -d '%')
if [ "$DISK" -gt 75 ]; then
  echo "Disk usage: ${DISK}%"
fi
```

---

## Alert Channels

| Severity | Channel              | Format                            |
| -------- | -------------------- | --------------------------------- |
| INFO     | Log only             | `[INFO] message`                  |
| WARNING  | Telegram             | `WARNING \| [type] \| [details]`  |
| CRITICAL | Telegram (immediate) | `CRITICAL \| [type] \| [details]` |

---

## Health Metrics

### Normal Operating Parameters

| Metric | Normal | Warning | Critical |
|--------|--------|---------|----------|
| Disk Usage | < 60% | 60-75% | > 75% |
| Context Window | < 100K | 100-150K | > 150K |
| Session Time | < 3hr | 3-4hr | > 4hr |
| Memory Usage | < 70% | 70-85% | > 85% |

---

## Recovery Procedures

### Disk Full
```
1. Stop active scans
2. Compress completed scan outputs
3. Move large files to external storage
4. Clear temporary files
5. Resume scans
```

### Context Overflow
```
1. Summarize completed phases
2. Reference file paths only
3. Clear raw output from context
4. Continue with reduced verbosity
```

### Session Timeout
```
1. Save current state to memory
2. Document completed phases
3. List pending phases
4. Provide handoff summary
```

---

## Monitoring Commands

### Quick Health Check
```bash
echo "Disk: $(df -h / | awk 'NR==2{print $5}')"
echo "Mem: $(free -h | awk 'NR==2{print $3"/"$2}')"
echo "Uptime: $(uptime -p)"
```

### Scan Output Size
```bash
du -sh workspace/scans/$TARGET/
```

### Active Processes
```bash
ps aux | grep -E 'subfinder|amass|nmap|nuclei|httpx' | grep -v grep
```
