# Treat transient recovery separately from lifecycle correctness

## Scenario and goal

With V-Shell and Dash to Dock enabled, one Super press should reliably open the overview, including after displays are connected, disconnected, or reconfigured.

## Wrong choice and consequence

Changing `panel-overview-style` fixed the visible transparent-panel symptom, and a second Super press let V-Shell repair its stale Dash overrides. Treating that recovered state as a complete fix missed the lifecycle defect. After another monitor reconfiguration, Dash to Dock rebuilt its Dash and the first overview transition failed again with NaN allocation warnings.

## Better approach and signals

Correlate the failure time with extension lifecycle events, not only the final appearance. Dash to Dock rebuilds its Dash on `monitors-changed`, changing `_workId`; V-Shell must debounce the display changes and repair its overrides after the new Dash exists, before the next overview animation begins. Keep the first-overview check only as a fallback.

## Boundary

The panel-style setting remains the correct fix for an intentionally transparent top bar. It cannot fix a stale object binding or a transition race. A UI working once after a retry proves recovery, not lifecycle correctness.

## Evidence and status

- Date: 2026-09-22.
- Observed: monitor layout changed at 13:31; Dash and tiling layout were rebuilt; the first affected overview at 14:13 emitted repeated NaN warnings and then V-Shell logged `Updating overrides`.
- Fix: local commit adds proactive repair after monitor changes and unlock.
- Status: source/build validation complete; runtime hot-plug verification requires loading the new code in a fresh GNOME Shell session.
