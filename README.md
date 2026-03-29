# Pop!_OS Shutdown with AudioRelay

## Summary
System was shutting down abruptly when using AudioRelay.

## Root Cause
Old Intel audio controller likely failing under load.

## Solution
Redirected audio output to NVIDIA HDMI device.

## Result
System stable, no more shutdowns.

## Full Analysis
See: docs/audiorelay-case.md
