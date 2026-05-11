# Front Door AI Sidecar

This add-on runs the governed front-door hybrid AI sidecar under Home Assistant OS.

It subscribes to Frigate MQTT events, processes only configured cameras, publishes retained MQTT/Home Assistant state, and can send informational Home Assistant mobile notifications. It does not publish command topics and does not operate locks, sirens, relays, lights, or security state.

## Runtime Data

- `/data/homelab-control-plane.toml`: generated runtime config
- `/data/telemetry.sqlite3`: automation telemetry journal
- `/data/vision_cloud_calls.jsonl`: local cloud-call ledger
- `/data/household_event_intelligence.sqlite3`: structured event-memory store, without raw images or video
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

## Frame Resolution

Version `0.1.4` resolves OpenAI images from Frigate tracked-object event ids rather than assuming every MQTT review id is directly fetchable as an event snapshot. Review messages use their detection/object ids where available; event messages can still use the event-shaped message id.

Snapshot fetches retry short-lived `404`, `409`, timeout, and invalid-image responses using `vision_frame_snapshot_retry_attempts` and `vision_frame_snapshot_retry_backoff_seconds`. The add-on publishes frame-resolution diagnostics without image bytes.

Manual read-only smoke test from the private repo:

```powershell
.\scripts\test_front_door_frame_resolution.ps1 -EventId "<frigate-event-id>"
```

## Household Event Intelligence

Version `0.1.5` adds Household Event Intelligence: a durable semantic layer over processed Frigate events, local face/classification signals, OpenAI summaries, AWS identity fallback status, and notification outcomes. It stores structured event facts in SQLite under `/data`; it does not store raw video, image bytes, provider request bodies, or secrets.

The add-on publishes retained MQTT/Home Assistant state for `last_12h_summary`, `today_delivery_status`, `unknown_visitor_count`, `last_unknown_event`, and `event_memory_status`.

The default digest is deterministic and local. Optional Ollama narrative summaries are disabled by default with `event_intelligence_llm_enabled: false`; if enabled, only structured event facts are sent to Ollama.

## Notification Detail View

Version `0.1.6` keeps mobile notifications short and adds a Home Assistant deep link so tapping an alert opens the front-door AI dashboard. The defaults are:

- `vision_notification_click_url: /front-door-ai`
- `vision_notification_message_max_chars: 160`

Use the dashboard and daily digest examples in the private repo under `configs/integrations/` as manual Home Assistant review/apply artifacts. Notifications remain informational only and include no action buttons.

## Frigate Backfill And Snapshot Candidates

Version `0.1.7` backfills the event-memory store from recent Frigate tracked-object events on add-on startup. The default backfill is memory-only for the last 12 hours: it updates dashboard digest entities, but it does not call OpenAI, AWS, or Home Assistant notification services for historical events.

The sidecar treats Frigate review ids and tracked-object event ids separately. Review messages are metadata unless they include detection/object ids; snapshot frames are resolved from actual Frigate event ids such as `frigate://front_door/<event-id>/snapshot.jpg`.

Frigate `box` values are normalized from `[x, y, width, height]` into the sidecar contract shape `[left, top, right, bottom]` before ROI gates run. The summary ROI remains less strict than the AWS identity ROI.

## Notification Quality And Signal Hygiene

Version `0.1.10` keeps phone notifications focused on useful household alerts. Expected provider diagnostics such as `aws:no_face`, location/ROI skips, frame-resolution skips, and local-only backfill are kept in dashboard/log diagnostics rather than rendered as `partial result` phone messages or routine digest provider failures.

Known resident walk-ups remain suppressed by default. Delivery alerts still route when there is stronger package/OpenAI evidence, but a local `Delivery` classification with a confident resident identity does not phone-alert by itself.

## Updating From Repo Changes

Home Assistant reads this add-on from this public metadata repo, not from the private source repo. A new version is available to Home Assistant only after:

1. the private source repo has built and published the GHCR image tag
2. this public add-on metadata repo has been updated to the same version

Home Assistant does not watch the private source repo commit or tag directly, and restarting Home Assistant is not the update mechanism. It discovers updates from the public add-on metadata repo and then pulls the matching public GHCR image.

For every sidecar tweak:

1. Push the private source repo and wait for GitHub Actions to publish `ghcr.io/bensturgess-austin/front-door-ai-sidecar:<version>`.
2. Confirm the GHCR tag exists and the package is public.
3. Push this public metadata repo with the same add-on `version`.
4. Wait a few minutes for Home Assistant Supervisor to refresh the add-on repository.
5. If the update does not appear, refresh or force the Home Assistant `update.front_door_ai_sidecar_update` entity with the target version.

If Supervisor returns `500 Internal Server Error` during a forced update, check the update entity before retrying. The update has worked when `installed_version` and `latest_version` both show the target version.

## Cutover

Run this add-on in parallel with the `newton` process first. Stop the `newton` background sidecar only after the add-on has processed a front-door event and Home Assistant mobile notification delivery is confirmed.
