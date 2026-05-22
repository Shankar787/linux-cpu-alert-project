
# Linux CPU Alert Project

## About
This is a Linux shell scripting project that monitors CPU usage continuously and sends alerts when CPU usage becomes high.

The script:
- Monitors CPU usage
- Sends email alerts
- Shows popup warnings
- Runs continuously in the background

---

## Technologies Used
- Linux
- Bash Shell Scripting
- awk
- cron
- zenity
- mailutils

---

## Features
- Warning alert at 90% CPU usage
- Critical alert at 95% CPU usage
- Email notification support
- Desktop popup notifications
- Popup cooldown protection
- Real-time CPU monitoring

---

## Script

```bash
PASTE YOUR SCRIPT HERE
```

---

## How to Run

Install required packages:

```bash
sudo apt install mailutils zenity
```

Make script executable:

```bash
chmod +x cpu_monitor.sh
```

Run the script:

```bash
./cpu_monitor.sh
```
#!/bin/bash

# =========================================================
# CPU Monitor Script
#
# I made this script for my Linux system to keep an eye on
# CPU usage. If the CPU usage gets too high, the script:
#
# 1. Sends an email alert
# 2. Shows a popup warning on the desktop
#
# Useful for monitoring system load during heavy tasks.
# =========================================================


# ---------- User Settings ----------

WARNING_THRESHOLD=90
CRITICAL_THRESHOLD=95

EMAIL_TO="testmail@.com"

# How often the script checks CPU usage
CHECK_INTERVAL_SECONDS=10

# Prevents popup spam
POPUP_COOLDOWN_SECONDS=300


# ---------- Internal Variables ----------

last_popup_time=0

# Stop script safely with CTRL+C
trap "echo -e '\nCPU monitor stopped.'; exit 0" SIGINT SIGTERM


# ---------- Function to Calculate CPU Usage ----------

get_current_cpu_usage() {

    # Read CPU stats from /proc/stat
    # Taking two readings helps calculate actual CPU activity

    local prev_stats=($(head -n1 /proc/stat))
    local prev_idle=${prev_stats[4]}

    local prev_total=0
    for val in "${prev_stats[@]:1}"; do
        prev_total=$((prev_total + val))
    done

    # Wait for 1 second before taking another reading
    sleep 1

    local curr_stats=($(head -n1 /proc/stat))
    local curr_idle=${curr_stats[4]}

    local curr_total=0
    for val in "${curr_stats[@]:1}"; do
        curr_total=$((curr_total + val))
    done

    # Difference between old and new values
    local total_diff=$((curr_total - prev_total))
    local idle_diff=$((curr_idle - prev_idle))

    # Avoid division by zero
    if [ "$total_diff" -eq 0 ]; then
        echo 0
    else
        local usage_percent=$((100 * (total_diff - idle_diff) / total_diff))
        echo "$usage_percent"
    fi
}


# ---------- Main Monitoring Loop ----------

echo "CPU monitor started..."
echo "Checking CPU usage every $CHECK_INTERVAL_SECONDS seconds."
echo "Press CTRL+C to stop."

while true; do

    cpu_now=$(get_current_cpu_usage)

    echo "Current CPU Usage: $cpu_now%"

    # ---------- Critical Alert ----------
    if [ "$cpu_now" -ge "$CRITICAL_THRESHOLD" ]; then

        message="CRITICAL ALERT: CPU usage is at $cpu_now% on $(hostname)."

        echo "$message" | mail -s "Critical CPU Alert" "$EMAIL_TO"

        current_time=$(date +%s)

        # Show popup only after cooldown period
        if [ $((current_time - last_popup_time)) -gt "$POPUP_COOLDOWN_SECONDS" ]; then

            zenity --error \
            --title="Critical CPU Alert" \
            --text="$message"

            last_popup_time=$current_time
        fi


    # ---------- Warning Alert ----------
    elif [ "$cpu_now" -ge "$WARNING_THRESHOLD" ]; then

        message="WARNING: CPU usage is at $cpu_now% on $(hostname)."

        echo "$message" | mail -s "CPU Usage Warning" "$EMAIL_TO"

        (
            zenity --warning \
            --title="CPU Warning" \
            --text="$message" \
            --ok-label="Dismiss"
        ) &
    fi


    # Wait before next check
    sleep "$CHECK_INTERVAL_SECONDS"

done
---

## Future Improvements
- Add log file support
- Add RAM monitoring
- Create systemd service
- Add Discord/Slack notifications
 ---
 
 ## Screenshots

### CPU Warning and Critical Alerts

<img width="1778" height="897" alt="CPU Alert Project" src="https://github.com/user-attachments/assets/7ad07e25-b755-46bd-9ad0-fb048d11d651" />
