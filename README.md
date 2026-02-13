# PyApple MCP Tools

[![PyPI Downloads](https://static.pepy.tech/personalized-badge/pyapple-mcp?period=total&units=INTERNATIONAL_SYSTEM&left_color=BLACK&right_color=GREEN&left_text=downloads)](https://pepy.tech/projects/pyapple-mcp)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)
[![MCP](https://img.shields.io/badge/MCP-compatible-purple.svg)](https://modelcontextprotocol.io)

A Python implementation of Apple-native tools for the [Model Context Protocol (MCP)](https://modelcontextprotocol.com/docs/mcp-protocol), providing seamless integration with macOS applications.

## Features

- **Messages**: Send and read messages using the Apple Messages app
- **Notes**: List, search, create, and delete notes in Apple Notes app  
- **Contacts**: Search contacts from Apple Contacts
- **Emails**: Send emails, search messages, and manage mail with Apple Mail
- **Reminders**: List, search, and create reminders in Apple Reminders app
- **Calendar**: Search events, create calendar entries, and manage your schedule
- **Web Search**: Search the web using DuckDuckGo
- **Maps**: Search locations, get directions, and manage guides with Apple Maps

## Quick Installation

### Using uv (Recommended)

[uv](https://docs.astral.sh/uv/) keeps pyapple-mcp in an isolated virtual environment so it never touches your system Python.

1. **Clone and set up the project**:
   ```bash
   git clone https://github.com/pyapple-mcp/pyapple-mcp.git
   cd pyapple-mcp
   uv venv
   uv pip install -e .
   ```

2. **Configure Claude Desktop** by editing `~/Library/Application Support/Claude Desktop/claude_desktop_config.json`:
   ```json
   {
     "mcpServers": {
       "pyapple": {
         "command": "/path/to/pyapple-mcp/.venv/bin/python",
         "args": ["-m", "pyapple_mcp.server"]
       }
     }
   }
   ```

   Replace `/path/to/pyapple-mcp` with the actual path to your cloned repository.

3. **Restart Claude Desktop** to load the new configuration.

> **Why point to `.venv/bin/python` directly?** The Mail and Calendar handlers read Apple SQLite databases directly, which requires Full Disk Access (see [Permissions Setup](#permissions-setup)). macOS grants FDA per-binary, so using a stable venv Python path lets you grant FDA once. If you use `uv run` instead, the underlying Python binary can change when uv updates, breaking your FDA grant.

### Alternative: uv run (No Persistent venv)

If you don't need Mail database access or have already handled FDA, you can skip the explicit venv and let `uv run` manage everything:

```json
{
  "mcpServers": {
    "pyapple": {
      "command": "uv",
      "args": [
        "run",
        "--directory",
        "/path/to/pyapple-mcp",
        "pyapple-mcp"
      ]
    }
  }
}
```

### Using pip (Not Recommended)

This installs directly into your active Python environment, which can cause dependency conflicts over time.

```bash
pip install pyapple-mcp
```

You can then run the setup helper to configure Claude Desktop automatically:

```bash
pyapple-mcp-setup
```

## Usage Examples

### Basic Commands

```
Can you send a message to John Doe saying "Hello from Claude!"?
```

```
Find all notes about "AI research" and summarize them
```

```
Create a reminder to "Buy groceries" for tomorrow at 5pm
```

```
Search my calendar for events this week containing "meeting"
```

```
Get directions from "Apple Park" to "San Francisco Airport"
```

### Advanced Workflows

You can chain commands together for complex workflows:

```
"Read my note about the conference attendees, find their contact information, and send them a thank you email"
```

## Development

### Local Setup

1. **Clone the repository**:
   ```bash
   git clone https://github.com/pyapple-mcp/pyapple-mcp.git
   cd pyapple-mcp
   ```

2. **Create a venv and install with dev dependencies**:
   ```bash
   uv venv
   uv pip install -e ".[dev]"
   ```

3. **Run the development server**:
   ```bash
   .venv/bin/python -m pyapple_mcp.server
   ```

### Testing with MCP Inspector

```bash
# Test the server
mcp dev pyapple_mcp/server.py

# Test with dependencies
mcp dev pyapple_mcp/server.py --with httpx --with beautifulsoup4
```

## Requirements

- **macOS 10.15+** (Catalina or later)
- **Python 3.10+**
- **Appropriate permissions** for accessing:
  - Contacts
  - Calendar
  - Messages
  - Mail
  - Notes
  - Reminders
  - Automation (for controlling apps)

## Permissions Setup

On first use, macOS will prompt for various permissions. Grant access to:

1. **Contacts** — for contact search functionality
2. **Calendar** — for calendar event management
3. **Messages** — for sending/reading messages
4. **Mail** — for email operations
5. **Notes** — for notes access
6. **Reminders** — for reminder management
7. **Automation** — for controlling applications via AppleScript

### Full Disk Access (Required for Mail and Calendar)

The Mail handler reads Apple Mail's SQLite database (`~/Library/Mail/V10/MailData/Envelope Index`) and `.emlx` message files directly for performance. The Calendar handler reads the Calendar SQLite database (`~/Library/Group Containers/group.com.apple.calendar/Calendar.sqlitedb`) directly. macOS protects both paths behind Full Disk Access (FDA).

To grant FDA:

1. Identify the Python binary the server is using:
   ```bash
   # If using the recommended venv setup:
   readlink -f /path/to/pyapple-mcp/.venv/bin/python

   # If using uv run:
   uv run --directory /path/to/pyapple-mcp python -c "import sys; print(sys.executable)"
   ```
2. Open **System Settings → Privacy & Security → Full Disk Access**.
3. Click **+**, press **Cmd+Shift+G**, paste the Python binary path, and add it.
4. Restart Claude Desktop.

> **Tip:** Granting FDA to Claude Desktop or your terminal app (e.g., iTerm2) may also propagate to child processes, though this is not guaranteed on all macOS versions.

## Configuration

You can customize pyapple-mcp behavior by creating a config file at `~/.config/pyapple-mcp/config.json`.

### Available Options

```json
{
  "excluded_calendars": ["Calendar Name 1", "Calendar Name 2"]
}
```

| Option | Type | Description |
|--------|------|-------------|
| `excluded_calendars` | array of strings | Calendar names to exclude from all queries |

## Troubleshooting

### Common Issues

**Permission Denied Errors**:
- Go to **System Settings > Privacy & Security**
- Grant access to the required applications
- Restart Claude Desktop

**Mail "Cannot access" or empty results**:
- This almost always means the Python binary lacks Full Disk Access. See [Permissions Setup](#full-disk-access-required-for-mail-and-calendar).
- If using `uv run`, the Python binary path may have changed after an update. Re-check with `uv run ... python -c "import sys; print(sys.executable)"` and re-grant FDA.

**Calendar "No calendars found" but Calendar.app has data**:
- This almost always means the Python binary lacks Full Disk Access. See [Permissions Setup](#full-disk-access-required-for-mail-and-calendar).
- If using `uv run`, the Python binary path may have changed after an update. Re-check with `uv run ... python -c "import sys; print(sys.executable)"` and re-grant FDA.

**Module Import Errors**:
- Ensure you're running on macOS
- Install PyObjC frameworks: `uv pip install pyobjc`

**AppleScript Execution Errors**:
- Check that the target applications are installed
- Verify automation permissions in System Settings

**Setup Issues**:
- Run `pyapple-mcp-setup --help` for setup options
- Check that pyapple-mcp is in your PATH: `which pyapple-mcp`
- Use `pyapple-mcp-setup --config-path /path/to/config` for custom config locations

### Debug Mode

Run with debug logging:
```bash
PYAPPLE_DEBUG=1 .venv/bin/python -m pyapple_mcp.server
```

## Architecture

```
pyapple-mcp/
├── pyapple_mcp/
│   ├── __init__.py
│   ├── server.py          # Main MCP server
│   ├── setup_helper.py    # Setup and configuration helper
│   └── utils/
│       ├── __init__.py
│       ├── applescript.py # AppleScript execution
│       ├── calendar.py    # Calendar integration
│       ├── contacts.py    # Contacts integration
│       ├── mail.py        # Mail integration
│       ├── maps.py        # Maps integration
│       ├── messages.py    # Messages integration
│       ├── notes.py       # Notes integration
│       ├── reminders.py   # Reminders integration
│       └── websearch.py   # Web search functionality
├── tests/
├── requirements.txt
├── README.md
├── LICENSE
└── pyproject.toml
```

## Contributing

1. Fork the repository
2. Create a feature branch: `git checkout -b feature-name`
3. Make your changes
4. Run tests: `pytest`
5. Submit a pull request

## License

MIT License - see [LICENSE](LICENSE) file for details.

## Acknowledgments

- Inspired by the original [apple-mcp](https://github.com/dhravya/apple-mcp) TypeScript implementation
- Built with the [MCP Python SDK](https://github.com/modelcontextprotocol/python-sdk)
- Uses PyObjC for macOS system integration 
