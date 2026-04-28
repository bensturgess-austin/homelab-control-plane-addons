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

## Cutover

Run this add-on in parallel with the `newton` process first. Stop the `newton` background sidecar only after the add-on has processed a front-door event and Home Assistant mobile notification delivery is confirmed.
