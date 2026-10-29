# HLT Forge deployment continuity

Checked 29 October 2026. This repository owns the original core Blueprint, not a new installation.

- Console: https://forge.katailyst.com
- API: https://dify-api-7585.onrender.com (health reports Dify 1.18.0)
- Blueprint: exs-da0ka5c9v7es739imsn0; auto-sync remains paused.
- API: srv-da0kbkdg1s2s73bs97rg; worker: srv-da0kbkdg1s2s73bs97qg; web: srv-da0kbl5g1s2s73bs9at0.
- Existing private plugin daemon: srv-da118uu7bikc7385ffog, name dify-plugin-daemon, image 0.6.10-local; 5 GB persistent disk.
- Existing scheduler: srv-dbhm0qbbc2fs73erfa7g, name dify-beat; starter worker with MODE=beat. It shares the API 1.18.0 digest and worker configuration, with MIGRATION_ENABLED=false and ENABLE_WORKFLOW_SCHEDULE_POLLER_TASK=true. No customer workflow schedule was enabled by this repair.
- Postgres: dpg-da0kb3tg1s2s73bs8kt0-a, PostgreSQL 16; existing databases dify_20be and dify_plugin. Redis: red-da0kb3lg1s2s73bs8kk0.

## Storage and credentials

API, worker and Beat use OpenDAL S3 and the existing private bucket `hlt-dify-forge-private`. Their endpoint, bucket, access key and secret values remain in Render/Doppler. The API's old 10 GB disk is retained as a historical recovery copy. It does not receive new shared-storage writes and is not a current rollback snapshot. Do not regenerate the application SECRET_KEY, delete tenant encryption keys, switch back to fs, or remove the old disk during routine sync.

The Blueprint uses `sync: false` for existing connection values. Render ignores these entries on an update and preserves the configured values. For a new install they require explicit configuration; this file alone does not provision the bucket, plugin database, plugin daemon, or scheduler. Do not click a gallery deploy button to repair this installation.

## Safe next upgrade

1. Inspect the existing Blueprint's proposed diff. Keep API, worker, web, database, Redis, plugin daemon and Beat identities. Stop on a replacement, deletion, database rename, credential reset or duplicate service. Keep CONSOLE_WEB_URL and APP_WEB_URL at https://forge.katailyst.com. Returning OAuth popups to the underlying Render web host crosses the console origin and breaks the callback UI after authorization.
2. Preserve database recovery and current environment values through the existing secret-safe backup path. Pin the official API/web image digests as a matched release; API, worker and Beat must use the same API image.
3. Upgrade the existing services, with migrations on API only. A Blueprint change is not a completed deployment. The plugin daemon and Beat are managed separately until explicitly imported and reviewed in the Blueprint; do not enable auto-sync as a shortcut.
4. Verify health, API and worker access to the same encryption key and uploaded file, Beat dispatch and worker receipt, a real model run, and K2's persisted result readback after reload. Health alone is insufficient.

## Useful workflow

`HLT — Source Brief & Action Plan` (app `5b4009bd-f047-473a-985f-63af1c45df39`) accepts `source_packet` and `goal`, returning `brief`. It is a private source-grounded workflow; the Service API is enabled and the public web app is disabled. K2 retains the app credential in its org vault and executes it through `tool:dify-forge-workflow`.

The shared storage, six completed runs and scheduler receipts are recorded at https://katailyst.com/board/ab441204-7080-48aa-9a2c-b87a05032ef6. Current continuation: https://katailyst.com/board/fdc9bf72-dff3-4871-a429-236d351db7cd.

## K2 connection and maintenance

Dify's native HTTP MCP provider has server identifier `katailyst` and endpoint https://katailyst.com/api/mcp. Alec approved its account-level operator access on 29 October 2026. This connection inherits the authorizing account's visibility, including personal content; it is not an organization-only service identity. Use the workflow's selected tools and existing K2 effect controls deliberately. Credentials remain in Dify's encrypted OAuth storage. Preserve the server identifier when importing app definitions.

The Codex heartbeat `keep-dify-healthy-and-current` checks on Mondays at 08:00 America/Chicago. It compares official stable releases and provider plugins, checks actual workflow behavior, and applies compatible pinned updates only with verified recovery. It reports meaningful upgrades or failures and stays quiet when unchanged. This is local scheduled maintenance, not an always-on hosted uptime monitor or blind Blueprint auto-update.
