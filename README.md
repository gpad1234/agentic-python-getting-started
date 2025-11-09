# Anthropic Python Project with MCP OSQuery Server

A Python project featuring:
1. **Claude API Integration** - Using Anthropic's Claude models
2. **MCP OSQuery Server** - System information queries via Model Context Protocol

## 🚀 Quick Start

1. **Activate the pre-configured environment:**
   ```bash
   source venv/bin/activate
   ```

2. **Set up your API key:**
   ```bash
   cp .env.example .env
   # Edit .env and add your Anthropic API key
   ```

3. **Run the project:**
   ```bash
   python main.py              # Test Claude API
   python demo_osquery_server.py  # Demo MCP capabilities
   ```

## 🔒 Security Setup

**IMPORTANT**: This project uses environment variables for API keys:

- ✅ **`.env`** - Contains your actual API keys (git ignored)
- ✅ **`.env.example`** - Safe template (can be committed)
- ✅ **`.gitignore`** - Protects all secrets from git

**Get your API key:**
- Visit [Anthropic Console](https://console.anthropic.com/)
- Create an account and generate an API key
- Add to `.env`: `ANTHROPIC_API_KEY=your_key_here`

## Usage

### Run Claude Example
```bash
python main.py
```

### Run MCP OSQuery Server Demo
```bash
python demo_osquery_server.py
```

### Start MCP OSQuery Server (requires osquery installed)
```bash
python -m mcp_osquery_server.server
```

## Project Structure

- `main.py` - Example script showing how to use the Anthropic API
- `mcp_osquery_server/` - MCP server for system queries
  - `server.py` - Main MCP server
  - `osquery_tools.py` - osquery wrapper functions
  - `README.md` - Detailed documentation
- `demo_osquery_server.py` - Demo showing MCP capabilities
- `requirements.txt` - Python dependencies
- `.env.example` - Template for environment variables
- `.env` - Your actual environment variables (not tracked in git)
- `SETUP_GUIDE.md` - Complete setup and integration guide

## Dependencies

- `anthropic` - Official Anthropic Python library
- `mcp` - Model Context Protocol library
- `python-dotenv` - Load environment variables from .env files
- `pydantic` - Data validation

## MCP OSQuery Server

The `mcp_osquery_server` provides system information queries through the MCP protocol. Available tools:

- `system_info` - Get system information
- `processes` - Get running processes
- `users` - Get system users
- `network_interfaces` - Get network adapters
- `network_connections` - Get active connections
- `open_files` - Get open files
- `disk_usage` - Get disk usage
- `installed_packages` - Get installed software
- `running_services` - Get running services
- `custom_query` - Execute custom osquery SQL

See `SETUP_GUIDE.md` for detailed integration instructions.

## Getting Started

1. **Follow the security setup above** 🔒
2. **Read `SETUP_GUIDE.md`** for complete documentation
3. **Run the demo**: `python demo_osquery_server.py`
4. **Test Claude**: `python main.py`

## 📁 Project Files

- `main.py` - Claude API example script
- `mcp_osquery_server/` - MCP server for system queries
  - `server.py` - Main MCP server (compatible with MCP 1.21.0)
  - `osquery_tools.py` - osquery wrapper functions
  - `README.md` - Detailed server documentation
- `demo_osquery_server.py` - Demo showing capabilities
- `requirements.txt` - Python dependencies
- `.env.example` - Environment template (safe)
- `.env` - Your API keys (git ignored)
- `.gitignore` - Comprehensive security rules
- `SETUP_GUIDE.md` - Complete setup guide
- `.vscode/tasks.json` - VS Code task configuration

## 🛡️ Security Features

- ✅ **Environment files protected** (`.env*` git ignored)
- ✅ **API keys secured** (never in source code)  
- ✅ **Virtual environment ignored** (`venv/` excluded)
- ✅ **Comprehensive gitignore** (secrets, credentials, keys)
- ✅ **Ready for production** (secure by default)

## ⚠️ Security Reminder

**Never commit these files:**
- `.env` (contains real API keys)
- Any `*.key`, `*.pem`, or credential files
- Virtual environment directories

The `.gitignore` is configured to prevent this automatically.