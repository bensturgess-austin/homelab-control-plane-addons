# Front Door AI Sidecar

This add-on runs the governed front-door hybrid AI sidecar under Home Assistant OS.

It subscribes to Frigate MQTT events, processes only configured cameras, publishes retained MQTT/Home Assistant state, and can send informational Home Assistant mobile notifications. It does not publish command topics and does not operate locks, sirens, relays, lights, or security state.

## Runtime Data

- `/data/homelab-control-plane.toml`: generated runtime config
- `/data/telemetry.sqlite3`: automation telemetry journal
- `/data/vision_cloud_calls.jsonl`: local cloud-call ledger
- `/share/front_door_ai/enrolled_household/`: default read-only location for enrolled resident reference images and manifest

## First Start

1. Install the add-on with `vision_dry_run_enabled: true`.
2. Configure MQTT broker details and `vision_frigate_base_url`.
3. Start the add-on and confirm the log shows subscriptions to `frigate/reviews` and `frigate/events`.
4. Add OpenAI/AWS keys only when ready to switch to `vision_provider_mode: hybrid`.
5. Enable notifications only after dry-run MQTT state is confirmed.

## Commissioning Notifications

Use `vision_commissioning_notifications_enabled: true` only while proving mobile notification delivery. Commissioning notifications are clearly labelled and should be turned off after a successful walk test.

Leave `vision_notify_delivery_always: true` enabled for normal operation so delivery/package events can notify even when one provider returns a partial or skipped result. Leave `vision_notify_known_residents: false` unless resident walk-up notifications are explicitly desired.

## Local Frigate Signals

Version `0.1.3` enriches front-door events from fresh Home Assistant state for local face recognition and person classification. The default freshness window is `vision_local_signal_freshness_seconds: 120`.

Fresh `Resident`, `Visitor`, and `Delivery` classifications can help summary/notification routing when Frigate event zone fields are sparse. AWS identity fallback remains stricter and still requires the configured identity zone or ROI gate.

## Cutover

Run this add-on in parallel with the `newton` process first. Stop the `newton` background sidecar only after the add-on has processed a front-door event and Home Assistant mobile notification delivery is confirmed.
