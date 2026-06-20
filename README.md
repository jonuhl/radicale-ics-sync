# radicale-ics-sync

Ever wanted to subscribe to an ICS feed (like a university timetable or a shared calendar) in [Radicale](https://radicale.org), but still be able to edit events locally? That's what this plugin is for.

It syncs external ICS feeds into your Radicale calendars, keeps your local changes intact, and lets you filter out events you don't care about.

## Installation

```bash
pip install radicale-ics-sync
```

Then configure Radicale to use the plugin (see [Configuration](#configuration)).

## Configuration

### Radicale config

In your Radicale `config` file, set the storage type to `radicale_ics_sync.storage`:

```ini
[storage]
type = radicale_ics_sync.storage
filesystem_folder = /data/collections
# ics_config is optional; if omitted, defaults to /config/ics_sync.json
ics_config = /config/ics_sync.json
```

### Sync jobs (`ics_sync.json`)

Create a file `/config/ics_sync.json` that defines which feeds to sync and where.

```json
[
  {
    "feed": "https://example.com/calendar.ics",
    "collection": "username/calendar-name",
    "sync_interval": 3600,
    "include_patterns": [],
    "exclude_patterns": []
  }
]
```
You can provide multiple pairs of feed + collection.
Each sync job supports these fields:

| Field | Required | Default | Description |
|---|---|---|---|
| `feed` | ✅ | — | URL of the ICS feed |
| `collection` | ✅ | — | Radicale collection path |
| `sync_interval` | | `3600` | How often to poll the feed, in seconds |
| `include_patterns` | | `[]` | Can be strings or arbitrary regex patterns |
| `exclude_patterns` | | `[]` | Can be strings or arbitrary regex patterns |

> **Note:** The collection must already exist in Radicale before the plugin can sync to it. Create it through the Radicale web interface or your CalDAV client first.

### Filtering

Patterns are matched case-insensitively against the event's `SUMMARY` (title) field.

```json
{
  "feed": "https://university.example.com/timetable.ics",
  "collection": "alice/uni",
  "exclude_patterns": ["Tutorial", "Exercise"]
}
```

- If `include_patterns` is set: only events matching at least one pattern are synced
- If `exclude_patterns` is set: events matching any pattern are removed

## Docker setup

Here is a minimal `docker-compose.yml`:

```yaml
services:
  radicale:
    build: .
    container_name: radicale
    restart: always
    ports:
      - "5232:5232"
    volumes:
      - ./config:/config/config:ro  # Radicale config file (not a directory)
      - ./users:/config/users:ro
      - ./ics_sync.json:/config/ics_sync.json:ro
      - ./data:/data
    read_only: true
    tmpfs:
      - /tmp
```

And a `Dockerfile` to install the plugin:

```dockerfile
FROM tomsquest/docker-radicale
RUN /venv/bin/pip install radicale-ics-sync --no-cache-dir
```

## Non-Docker setup
 
Install Radicale and the plugin:
 
```bash
pip install radicale radicale-ics-sync
```
 
Create your Radicale config (e.g. `~/.config/radicale/config`) and specify the path of your ics_sync.json:
 
```ini
[storage]
type = radicale_ics_sync.storage
filesystem_folder = ~/.local/share/radicale/collections
ics_config = ~/.config/radicale/ics_sync.json
```
 
Then create `~/.config/radicale/ics_sync.json` and start Radicale:
 
```bash
radicale
```

## State

The plugin stores its sync state in `ics_sync_hashes.json`, written next to your Radicale data. It tracks the last known content hash per event so unchanged events aren't rewritten. Safe to delete if you want a full re-sync on next startup.

## Behavior

The plugin polls ICS feeds at a regular interval. Events are filtered by include/exclude patterns before being written to Radicale. Events that disappear from the upstream feed or are filtered out are deleted from the Radicale collection. Local edits to upstream events are preserved, until the upstream event itself changes, in which case the upstream version wins. Local events are not touched.

## Limitations & Roadmap

- **Filtering is `SUMMARY`-only** — other fields (`LOCATION`, `DESCRIPTION`) aren't supported yet, but are planned.
- **Upstream changes fully overwrite local edits** — field-level merge (e.g. keeping your local title while accepting upstream time changes) is planned.

## Contributing

Issues and pull requests are welcome.

## License

MIT