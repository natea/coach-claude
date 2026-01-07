<div align="center">

<img src="https://raw.githubusercontent.com/ClutchEngineering/coach-claude/main/docs/flexed-biceps-right.png" width="80" alt="💪" /><img src="https://raw.githubusercontent.com/ClutchEngineering/coach-claude/main/docs/mechanical-arm.png" width="80" alt="🦾" />

# Coach Claude

</div>

A health and wellness system for developers that integrates seamlessly with Claude Code. Coach Claude helps you stay hydrated and active during coding sessions by suggesting water breaks and stretches during natural pauses in your workflow.

## Quick Start

```bash
# Install using uv (recommended)
uv tool install coach-claude

# Or with pipx (alternative)
pipx install coach-claude

# Setup (installs daemon and skill)
coach-claude install

# Enable in your project (run in each project directory)
claude mcp add coach-claude -t sse http://localhost:8787/sse

# Done! Open Claude Code and start chatting.
```

> **Note**: We recommend `uv` over `pipx` as it's faster and more reliable.
>
> **Install uv:**
> - **NixOS/nix-darwin** (recommended): Add `uv` to your packages. See [nixos-config](https://github.com/natea/nixos-config) for an example.
> - **Manual**: `curl -LsSf https://astral.sh/uv/install.sh | sh`

Visit http://localhost:8787/ for your dashboard.

<img src="https://raw.githubusercontent.com/ClutchEngineering/coach-claude/main/docs/webui.png" width="500" alt="Coach Claude web dashboard" />

## Features

- **Smart Reminders**: Automatic check-ins at conversation start and during long sessions
- **Non-Intrusive**: Suggestions appear during processing time, not when you're actively working
- **Bedtime Countdown**: Visual progress bar showing time until your target bedtime
- **Customizable**: Adjust thresholds, units, bedtime, and goals to match your preferences
- **Local-First**: All data stays on your machine in a SQLite database
- **Works Everywhere**: Run as a daemon on your host machine, accessible from any devcontainer
- **Web Dashboard**: View your stats at `http://localhost:8787/`

## How It Works

<img src="https://raw.githubusercontent.com/ClutchEngineering/coach-claude/main/docs/example-conversation.png" width="600" alt="Example conversation showing Coach Claude in action" />

## Commands

```bash
coach-claude install    # Full setup (daemon + skill + MCP config)
coach-claude uninstall  # Remove daemon
coach-claude start      # Start daemon
coach-claude stop       # Stop daemon
coach-claude restart    # Restart daemon
coach-claude status     # Show current status
coach-claude run        # Run server directly (without daemon)
```

## Manual Configuration

MCP servers are configured per-project in Claude Code. Run this in each project directory:

**For local use:**

```bash
claude mcp add coach-claude -t sse http://localhost:8787/sse
```

**For devcontainers** (run server on host machine):

```bash
claude mcp add coach-claude -t sse http://host.docker.internal:8787/sse
```

## MCP Tools

The server exposes these tools to Claude:

| Tool                                         | Description                                         |
| -------------------------------------------- | --------------------------------------------------- |
| `log_water`                                  | Log water intake (amount, unit, notes, minutes_ago) |
| `log_workout`                                | Log a workout (type, name, duration, minutes_ago)   |
| `check_reminders`                            | Check what reminders are due                        |
| `get_stats`                                  | Get statistics (today, week, month)                 |
| `get_config` / `update_config`               | View/change settings                                |
| `list_recent_water` / `list_recent_workouts` | List recent logs                                    |
| `delete_water` / `delete_workout`            | Delete log entries                                  |

## Configuration

Default settings:

| Setting                     | Default | Description                      |
| --------------------------- | ------- | -------------------------------- |
| `water_threshold_minutes`   | 60      | Minutes before water reminder    |
| `workout_threshold_minutes` | 120     | Minutes before workout reminder  |
| `water_unit`                | oz      | Display unit for water           |
| `daily_water_goal`          | 64      | Optional daily goal              |
| `bedtime_hour`              | 22      | Target bedtime hour (24h format) |
| `bedtime_minute`            | 0       | Target bedtime minute            |
| `wakeup_hour`               | 7       | Wake up hour (24h format)        |
| `wakeup_minute`             | 0       | Wake up minute                   |

Configure via Claude: "Set my water threshold to 45 minutes" or "Set my bedtime to 11pm"

## Development

```bash
# Clone repository
git clone https://github.com/ClutchEngineering/coach-claude.git
cd coach-claude

# Install in development mode
cd packages/mcp-server
pip install -e ".[dev]"

# Run tests
pytest

# Format code
black src/
ruff check src/
```

### Iteration Workflow

When making changes to the server, use this workflow to test:

```bash
cd packages/mcp-server
uv tool install . --force && coach-claude install && coach-claude restart
```

Then reconnect Claude Code to the MCP server with `/mcp` in your chat.

## Release

To publish a new version:

1. Update version in `packages/mcp-server/pyproject.toml`
2. Commit and tag: `git tag v1.0.1`
3. Push: `git push origin main --tags`
4. GitHub Actions will automatically publish to PyPI

## Backup

Coach Claude stores your health data locally. Back up regularly to avoid losing your history.

### Backup Script

Create a backup script at `~/bin/backup-ai-configs.sh`:

```bash
#!/bin/bash
# Backup script for Coach Claude and AI configuration files
# Run daily via cron or launchd

set -e

BACKUP_DIR="${BACKUP_DIR:-$HOME/Backups/ai-configs}"
DATE=$(date +%Y-%m-%d)
BACKUP_PATH="$BACKUP_DIR/$DATE"

mkdir -p "$BACKUP_PATH"

echo "Backing up AI configurations to $BACKUP_PATH..."

# Coach Claude database and logs
if [ -d "$HOME/.coach-claude" ]; then
    cp -r "$HOME/.coach-claude" "$BACKUP_PATH/coach-claude"
    echo "✓ Backed up ~/.coach-claude"
fi

# Personal AI Infrastructure config (skills, hooks, .env)
if [ -d "$HOME/.config/pai" ]; then
    cp -r "$HOME/.config/pai" "$BACKUP_PATH/pai"
    echo "✓ Backed up ~/.config/pai"
fi

# Claude Code config (MCP servers, settings, skills)
if [ -d "$HOME/.claude" ]; then
    cp -r "$HOME/.claude" "$BACKUP_PATH/claude"
    echo "✓ Backed up ~/.claude"
fi

# Compress the backup
cd "$BACKUP_DIR"
tar -czf "ai-configs-$DATE.tar.gz" "$DATE"
rm -rf "$DATE"

# Clean up backups older than 30 days
find "$BACKUP_DIR" -name "ai-configs-*.tar.gz" -mtime +30 -delete

echo "✓ Backup complete: $BACKUP_DIR/ai-configs-$DATE.tar.gz"
```

Make it executable:

```bash
chmod +x ~/bin/backup-ai-configs.sh
```

### Automated Backups (macOS)

Create a LaunchAgent to run backups daily:

```bash
cat > ~/Library/LaunchAgents/com.user.backup-ai-configs.plist << 'EOF'
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.user.backup-ai-configs</string>
    <key>ProgramArguments</key>
    <array>
        <string>/bin/bash</string>
        <string>-c</string>
        <string>$HOME/bin/backup-ai-configs.sh</string>
    </array>
    <key>StartCalendarInterval</key>
    <dict>
        <key>Hour</key>
        <integer>2</integer>
        <key>Minute</key>
        <integer>0</integer>
    </dict>
    <key>StandardOutPath</key>
    <string>/tmp/backup-ai-configs.log</string>
    <key>StandardErrorPath</key>
    <string>/tmp/backup-ai-configs.log</string>
</dict>
</plist>
EOF

launchctl load ~/Library/LaunchAgents/com.user.backup-ai-configs.plist
```

### What Gets Backed Up

| Directory | Contents |
|-----------|----------|
| `~/.coach-claude/` | SQLite database with water/workout history, logs |
| `~/.config/pai/` | PAI skills, hooks, .env with API keys |
| `~/.claude/` | Claude Code MCP configs, custom skills, settings |

### Secrets Management

The `.env` files contain sensitive API keys. Don't commit them to version control. Instead, use one of these approaches:

#### Option 1: 1Password (Recommended)

Store API keys in 1Password and fetch them at runtime:

```bash
# Store your keys in 1Password, then reference them:
export GOOGLE_API_KEY=$(op read "op://Private/Google API Key/credential")
export OPENAI_API_KEY=$(op read "op://Private/OpenAI API Key/credential")

# Or use 1Password's CLI integration with .env files:
# Create .env.template with references:
# GOOGLE_API_KEY=op://Private/Google API Key/credential
# Then inject at runtime:
op inject -i .env.template -o .env
```

Enable 1Password CLI integration:
1. Open 1Password → Settings → Developer → "Integrate with 1Password CLI"
2. Install CLI: `brew install 1password-cli`

#### Option 2: NixOS (agenix/sops-nix)

For NixOS users, use [agenix](https://github.com/ryantm/agenix) or [sops-nix](https://github.com/Mic92/sops-nix):

```nix
# Using agenix
age.secrets.pai-env = {
  file = ./secrets/pai-env.age;
  path = "/home/user/.config/pai/.env";
  owner = "user";
};

# Using sops-nix
sops.secrets."pai/env" = {
  sopsFile = ./secrets/pai.yaml;
  path = "/home/user/.config/pai/.env";
};
```

#### Option 3: Encrypted Backup

If backing up `.env` files, encrypt them:

```bash
# Encrypt with age (recommended)
age -p -o backup.tar.gz.age backup.tar.gz

# Or with GPG
gpg -c backup.tar.gz
```

### Continuous Sync with Syncthing (Recommended)

For real-time backup across devices, use [Syncthing](https://syncthing.net/):

```bash
# Install via Nix (recommended) - see https://github.com/natea/nixos-config
# Or manually:
brew install syncthing  # macOS
# apt install syncthing  # Linux
```

Configure Syncthing to sync these directories:

| Directory | Sync Strategy |
|-----------|---------------|
| `~/.coach-claude/` | Sync to all devices (health data) |
| `~/.config/pai/` | Sync to all devices (exclude `.env` - use 1Password) |
| `~/.claude/` | Sync to all devices (skills, settings) |

**Syncthing benefits:**
- Real-time sync across all your machines
- Decentralized - no cloud service required
- Conflict resolution built-in
- Version history with staggered file versioning

**Setup tips:**
1. Add folders via Syncthing web UI at `http://localhost:8384`
2. Use `.stignore` to exclude sensitive files:
   ```
   # .stignore in ~/.config/pai/
   .env
   *.log
   ```
3. Enable "File Versioning" → "Staggered" for point-in-time recovery

See [nixos-config](https://github.com/natea/nixos-config) for a NixOS/nix-darwin Syncthing configuration example.

### Manual Backup

Quick one-liner for an immediate snapshot:

```bash
tar -czf ~/ai-configs-backup-$(date +%Y%m%d).tar.gz \
    ~/.coach-claude ~/.config/pai ~/.claude 2>/dev/null
```

### Restore

```bash
# From Syncthing: Files are already synced - just install coach-claude on new machine

# From tar backup:
tar -xzf ~/ai-configs-backup-20240115.tar.gz -C /tmp
coach-claude stop
cp -r /tmp/.coach-claude ~/
coach-claude start
```

## License

Apache 2.0 - See [LICENSE](LICENSE) file for details.

## Support

- **Issues**: https://github.com/ClutchEngineering/coach-claude/issues
- **Email**: support@clutch.engineering
