# Front Door AI Sidecar

Front Door AI Sidecar is a Home Assistant OS add-on that listens to Frigate MQTT events, enriches front-door activity, publishes retained MQTT/Home Assistant state, and can send informational mobile notifications.

The add-on is observe-only. It does not publish command topics and does not control locks, lights, sirens, relays, alarm state, or other Home Assistant actions beyond optional mobile notifications.

## Required Configuration

Configure these values before enabling live operation:

- `vision_provider_mode`: start with `local_only` for zero-cloud operation, or `hybrid` for local signals plus gated OpenAI.
- `vision_dry_run_enabled`: set to `false` for real local processing.
- `vision_frigate_base_url`: Frigate HTTP URL, for example `http://<frigate-host>:5000`.
- `vision_mqtt_enabled`: set to `true` when MQTT publishing/subscription is configured.
- `vision_mqtt_broker_host`: MQTT broker host or IP.
- `vision_mqtt_username` / `vision_mqtt_password`: MQTT credentials if required.
- `vision_notification_services`: optional allowlisted Home Assistant notify services.
- `vision_notification_allowed_services`: must include every notify service the add-on is allowed to call.

Provider keys are optional and should be entered only if you deliberately enable those providers:

- `openai_api_key`: required for `openai_summary` or gated `hybrid` OpenAI calls.
- `aws_access_key_id`, `aws_secret_access_key`, and `vision_aws_region`: required only if AWS identity fallback is explicitly enabled.

Reference images, if used, should live outside git under the configured `vision_identity_enrollment_manifest_path`.

## Operating Profiles

**Zero-cloud trial**

```yaml
vision_provider_mode: local_only
vision_dry_run_enabled: false
vision_aws_identity_enabled: false
vision_openai_value_gate_enabled: true
```

This keeps event memory, digests, MQTT state, dashboard entities, local recognition/classification enrichment, and notification routing active without OpenAI or AWS calls.

**Tightly gated OpenAI**

```yaml
vision_provider_mode: hybrid
vision_dry_run_enabled: false
vision_aws_identity_enabled: false
vision_openai_value_gate_enabled: true
```

OpenAI is used only when the value gate passes, such as unknown visitor, after-hours unknown visitor, delivery/package evidence, medium/high risk, or explicit commissioning.

**Commissioning**

Enable commissioning only for a short manual test window, then disable it again:

```yaml
vision_commissioning_notifications_enabled: true
vision_openai_commissioning_enabled: true
```

## Dashboard Link

Mobile notifications deep-link to the dashboard path configured by `vision_notification_click_url`, default `/front-door-ai`. Create or import a Home Assistant dashboard at that path if you want notification taps to open a readable detail view.

## Updating

The add-on uses the public image:

```text
ghcr.io/bensturgess-austin/front-door-ai-sidecar
```

Home Assistant preserves configured add-on options during normal version updates. After updating, restart the add-on and check the add-on log for MQTT subscription and event-processing messages.

## Support Boundary

This public add-on package contains only the install metadata and the runtime container image reference. Deployment-specific values, secrets, reference images, private automation notes, and local operational scripts are intentionally not included.
