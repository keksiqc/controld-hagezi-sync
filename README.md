# ControlD Hagezi Sync

Automatically sync [Hagezi DNS blocklists](https://github.com/hagezi/dns-blocklists) to your [ControlD](https://controld.com/) profiles via the ControlD API.

[![Sync Hagezi to ControlD](../../actions/workflows/sync.yml/badge.svg)](../../actions/workflows/sync.yml)

## What it does

- Downloads the latest Hagezi blocklist folder definitions (JSON)
- Deletes existing folders in your ControlD profiles (by PK)
- Recreates them with fresh rules, batched in groups of 500
- Supports multiple profiles with different folder combinations
- Runs on a schedule or on-demand via GitHub Actions

## Quick Start (GitHub Actions)

1. **Fork or use this repo** as a template.
2. **Copy the config:**
   ```bash
   cp config.toml.example config.toml
   ```
3. **Edit `config.toml`** with your ControlD profile names and desired folder mappings.
4. **Commit `config.toml`** to the repo (do **not** put your API token in it).
5. **Add your API token** as a GitHub secret:
   - Go to **Settings → Secrets and variables → Actions → New repository secret**
   - Name: `CONTROLD_API_TOKEN`
   - Value: your ControlD API Write Token from [controld.com/dashboard/api](https://controld.com/dashboard/api)
6. **Run it:**
   - Go to **Actions → Sync Hagezi to ControlD → Run workflow**
   - Or wait for the daily 03:00 UTC cron job

## Quick Start (Local / Self-hosted)

```bash
# Clone
git clone https://github.com/0x11DFE/controld-hagezi-sync.git
cd controld-hagezi-sync

# Install dependencies
# Debian/Ubuntu: sudo apt install curl jq
# macOS:          brew install curl jq
# Termux:         pkg install curl jq

# Copy and edit config
cp config.toml.example config.toml
vim config.toml   # or nano, etc.

# Set your token (or add it to config.toml [settings])
export CONTROLD_API_TOKEN="your_token_here"

# Run
chmod +x sync-hagezi.sh
./sync-hagezi.sh
```

## Configuration Reference

All behavior is driven by `config.toml`.

| Section | Key | Description |
|---------|-----|-------------|
| `[settings]` | `api_token` | ControlD API Write Token. Prefer `CONTROLD_API_TOKEN` env var. |
| `[settings]` | `dry_run` | Set to `true` to preview without changes. |
| `[profiles]` | `names` | Array of exact ControlD profile names to sync. |
| `[folders]` | `"Name"` | Maps a friendly folder name to its Hagezi JSON URL. |
| `[profile_folders]` | `<profile>` | Array of folder names to sync to that profile. |

### Example: Adding a new profile

```toml
[profiles]
names = ["Tesla", "Kids", "Friends", "Adults", "Work"]

[profile_folders]
Work = ["Badware Hoster", "Most Abused TLDs"]
```

### Example: Adding a custom folder

```toml
[folders]
"My Custom List" = "https://example.com/my-folder.json"

[profile_folders]
Tesla = ["Badware Hoster", "My Custom List"]
```

## CLI Options

```bash
./sync-hagezi.sh [OPTIONS]

  --config FILE   Use a custom configuration file (default: config.toml)
  --dry-run       Preview changes without modifying ControlD
  --profile NAME  Sync only one profile
  -h, --help      Show help
```

### Examples

```bash
# Sync everything
./sync-hagezi.sh

# Preview changes for the "Tesla" profile
./sync-hagezi.sh --profile Tesla --dry-run

# Use a different config file
CONFIG_FILE=prod.toml ./sync-hagezi.sh
```

## GitHub Action Inputs

When running manually via **Actions → Run workflow**, you can specify:

| Input | Description |
|-------|-------------|
| `profile` | Sync only a specific profile (leave empty for all) |
| `dry_run` | Check the box to run in preview mode |

## Security Notes

- **Never commit `config.toml` if it contains your API token.** The `.gitignore` ignores `config.toml` by default, but if you override it, be careful.
- **Use GitHub Secrets** for the token in CI/CD.
- The script strips a leading `Bearer ` prefix from the token automatically if present.

## Requirements

- `bash` 4.3+
- `curl`
- `jq`

## How it works

1. Reads `config.toml` to know which profiles and folders to manage.
2. Fetches your ControlD profile list to resolve names to IDs.
3. Downloads each Hagezi folder JSON once (cached per run).
4. For each profile, deletes existing folders by PK, then recreates them with fresh rules.
5. Rules are inserted in batches of 500 to stay within API limits.
6. Prints a freshness report showing when each Hagezi list was last updated on GitHub.

## Troubleshooting

| Issue | Solution |
|-------|----------|
| `Missing dependencies` | Install `curl` and `jq`. |
| `Profile not found by name` | Ensure the profile name in `config.toml` matches exactly (case-sensitive) in ControlD. |
| `Failed to fetch profiles (HTTP 401)` | Your API token is invalid or expired. Generate a new one from the ControlD dashboard. |
| `Batch X failed (HTTP 4xx/5xx)` | Usually transient. Re-run the workflow. If persistent, check ControlD API status. |

## License

MIT — see [LICENSE](LICENSE)
