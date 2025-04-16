
# `journalctl` Cheat Sheet

## What is `journalctl`?
`journalctl` is a command-line utility used to query and display messages from the journal, managed by `systemd-journald`.

---

## Basic Commands

### Show All Logs
```bash
journalctl
```

### Show Logs for a Specific Unit
```bash
journalctl -u <unit_name>.service
```

### Show Logs for a Specific Boot
```bash
journalctl -b
```

### Show Logs from Previous Boot
```bash
journalctl -b -1
```

### Follow Logs (like `tail -f`)
```bash
journalctl -f
```

### Show Logs for a Specific Time Range
```bash
journalctl --since "2024-04-01 12:00:00" --until "2024-04-01 14:00:00"
```

---

## Filtering and Searching

### Show Logs for a Specific Priority
```bash
journalctl -p err
```

### Show Logs for a Specific Process ID
```bash
journalctl _PID=1234
```

### Grep-like Filtering
```bash
journalctl | grep "pattern"
```

---

## Output Options

### Show Output in Reverse Order (newest first)
```bash
journalctl -r
```

### Limit Number of Lines
```bash
journalctl -n 50
```

### Export Logs to a Text File
```bash
journalctl > logs.txt
```

---

## Maintenance

### Vacuum Old Logs (by time or size)
```bash
journalctl --vacuum-time=7d
journalctl --vacuum-size=500M
```

---

## Tips
- Use `TAB` completion for unit names.
- Combine with `watch` for live monitoring: `watch -n 1 journalctl -u your-service.service -n 10`
