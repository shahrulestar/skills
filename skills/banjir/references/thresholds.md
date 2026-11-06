# Flood levels and alert rules

How to read water-level thresholds, when to message, and what to say.

## Rules

- Stay silent while the station is normal. No messages, check-ins, or summaries. Silence means safe.
- Alert only at Alert (yellow) or higher.
- Send one all-clear when it drops back to normal, then go quiet. This is optional.
- One scheduled check per station, every 15 minutes, and never more often.
- Never invent or estimate a reading. Use only values read from the live page.
- Start every alert with the station name.

## JPS levels

| Status | Colour | What the agent does |
| --- | --- | --- |
| Normal | Below Alert | Nothing |
| Alert (Waspada) | Yellow | Send one alert |
| Warning (Amaran) | Orange | Send one alert when it reaches this band |
| Danger (Bahaya) | Red | Send one alert when it reaches this band |

Do not repeat an alert while the level stays in the same band.

Every station has its own metre values. Read them from the page each time. Never hard-code them.

## Council dashboards

- **MBSJ PAIR:** colour bands per station, yellow (Be Alert) and red (Danger). Read the band values from the station page.
- **MBSA i-LenSA:** river levels with siren triggers at warning and danger. Thresholds are on the map or the station detail.

## Reading the lines

- Yellow or Alert is where the watch starts.
- Red or Danger is critical.
- On some stations measured in metres above sea level (mASL), the page's Normal line works as the yellow line. Check the page's own labels.
- If the page shows four levels (Normal, Alert, Warning, Danger), alert from Alert upward.
- Always quote the source's own numbers, both when confirming and in alerts.

## Stale readings

- Only alert on a fresh reading. Check the Last Updated time before you compare levels.
- If the reading is more than 1 hour old, or the page is down, send "No fresh reading since [time]" once. Do not repeat it on every run. Keep checking quietly.

## Alert format

Short, calm, and plain. Colour, station, level, time, red line, link.

```text
🟡 [Station] ([River]): YELLOW. 2.85 m at 14:30 (red line 3.40 m). [source link]
🟠 [Station] ([River]): WARNING. 3.15 m at 15:15 (red line 3.40 m). [source link]
🔴 [Station] ([River]): RED/DANGER. 3.52 m at 16:00 (red line 3.40 m). [source link]
🟢 [Station] ([River]): Back to normal. 2.10 m at 21:15. [source link]
```

The green all-clear is optional and sent only once. If the source is not on the allowed list, add "(unofficial source)" before the link.

## Setup check

Before you create the scheduled check:

1. Fetch the current reading.
2. State the Alert (yellow) and Danger (red) lines from the source.
3. Ask the user:

```text
Watching [Station] every 15 min. I'll message you only if it reaches [Alert line] m. Now [level] m at [time]. OK?
```

Create the watch only after the user says yes.
