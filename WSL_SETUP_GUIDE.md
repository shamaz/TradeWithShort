# Step-by-Step Guide: Running Trading with Short in WSL

This guide will help you set up and run the trading simulation application in Windows Subsystem for Linux (WSL).

## Prerequisites

- Windows 10/11 with WSL installed
- WSL 2 distribution (Ubuntu recommended)

---

## Step 1: Install WSL (if not already installed)

If you don't have WSL installed, open PowerShell as Administrator and run:

```powershell
wsl --install
```

Restart your computer when prompted. After restart, WSL will complete the installation.

---

## Step 2: Open WSL Terminal

1. Open your WSL distribution (Ubuntu) from the Start menu, or
2. Open PowerShell/Command Prompt and type `wsl`

---

## Step 3: Update System Packages

```bash
sudo apt update && sudo apt upgrade -y
```

---

## Step 4: Install Python and Required System Dependencies

```bash
sudo apt install -y python3 python3-pip python3-venv build-essential curl
```

Verify Python installation:
```bash
python3 --version
```

---

## Step 5: Install uv (Python Package Manager)

The project uses `uv` for dependency management. Install it:

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

Add uv to your PATH (add this to your `~/.bashrc` or `~/.zshrc`):
```bash
source $HOME/.cargo/env
```

Or reload your shell:
```bash
source ~/.bashrc
```

Verify uv installation:
```bash
uv --version
```

---

## Step 6: Install Node.js and npm

The project uses `npx` for MCP servers. Install Node.js:

```bash
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs
```

Verify installation:
```bash
node --version
npm --version
```

---

## Step 7: Navigate to Project Directory

From Windows, your project is located at `F:\repos\trading_with_short`. In WSL, this path is accessible as:

```bash
cd /mnt/f/repos/trading_with_short
```

**Note:** WSL mounts Windows drives under `/mnt/` with lowercase drive letters.

---

## Step 8: Create .env File

Create a `.env` file in the project root with your API keys:

```bash
nano .env
```

Add the following environment variables (replace with your actual API keys):

```env
# OpenAI API Key (required)
OPENAI_API_KEY=your_openai_api_key_here

# Optional: Other model API keys (if using USE_MANY_MODELS=true)
DEEPSEEK_API_KEY=your_deepseek_api_key_here
GOOGLE_API_KEY=your_google_api_key_here
GROK_API_KEY=your_grok_api_key_here
OPENROUTER_API_KEY=your_openrouter_api_key_here

# Polygon API Key (optional - for real market data)
POLYGON_API_KEY=your_polygon_api_key_here
POLYGON_PLAN=free
# Options: "free", "paid", "realtime"

# Brave Search API Key (optional - for researcher)
BRAVE_API_KEY=your_brave_api_key_here

# Trading Configuration
RUN_EVERY_N_MINUTES=60
RUN_EVEN_WHEN_MARKET_IS_CLOSED=false
RESET_TRADERS=true
USE_MANY_MODELS=false
```

Save and exit (Ctrl+X, then Y, then Enter).

---

## Step 9: Install Python Dependencies with uv

The project uses `uv` for dependency management. Install dependencies:

```bash
uv sync
```

This will create a virtual environment and install all required packages.

---

## Step 10: Verify Database Directory

Ensure the `memory` directory exists (it should already exist):

```bash
ls -la memory/
```

If it doesn't exist, create it:
```bash
mkdir -p memory
```

---

## Step 11: Test the Setup

### Option A: Run Trading Floor (Main Application)

Start the trading floor scheduler:

```bash
uv run trading_floor.py
```

This will start the trading simulation that runs every N minutes (default: 60 minutes).

### Option B: Run Gradio UI

In a new terminal window (or use `tmux`/`screen` for multiple sessions), start the Gradio interface:

```bash
cd /mnt/f/repos/trading_with_short
uv run app.py
```

The Gradio interface will start and open in your default browser. The URL will be displayed in the terminal (typically `http://127.0.0.1:7860`).

---

## Step 12: Running Multiple Services (Recommended)

The application requires multiple services to run:

1. **Trading Floor** - Main scheduler that runs traders
2. **Gradio UI** - Web interface for monitoring

### Using tmux (Recommended)

Install tmux:
```bash
sudo apt install -y tmux
```

Start a tmux session:
```bash
tmux new -s trading
```

Split the window:
- Press `Ctrl+B`, then `%` to split vertically
- Press `Ctrl+B`, then `"` to split horizontally

In each pane:
1. First pane: `uv run trading_floor.py`
2. Second pane: `uv run app.py`

Detach from tmux: `Ctrl+B`, then `d`
Reattach: `tmux attach -s trading`

### Using screen

```bash
sudo apt install -y screen
screen -S trading
```

Split using screen's commands or use separate screen sessions.

### Using Separate Terminal Windows

Simply open multiple WSL terminal windows and run each command in a separate window.

---

## Step 13: Accessing the Application

### From Windows Browser

The Gradio interface runs on `http://127.0.0.1:7860` or `http://localhost:7860`. You can access it directly from your Windows browser.

### From WSL Browser (if needed)

If you need to access from within WSL, you may need to set up X11 forwarding or use WSLg (Windows 11).

---

## Troubleshooting

### Issue: "uv: command not found"

**Solution:** Make sure you've added uv to your PATH:
```bash
source $HOME/.cargo/env
source ~/.bashrc
```

### Issue: "npx: command not found"

**Solution:** Verify Node.js is installed:
```bash
node --version
npm --version
```

If not installed, reinstall Node.js (see Step 6).

### Issue: Database errors

**Solution:** Ensure you have write permissions in the project directory:
```bash
chmod -R u+w /mnt/f/repos/trading_with_short
```

### Issue: API Key errors

**Solution:** 
1. Verify your `.env` file exists and contains valid API keys
2. Check that the `.env` file is in the project root directory
3. Ensure there are no extra spaces or quotes around API key values

### Issue: Port already in use

**Solution:** If port 7860 is already in use, Gradio will automatically use the next available port. Check the terminal output for the actual URL.

### Issue: Permission denied errors

**Solution:** If you encounter permission issues with Windows files, you may need to:
```bash
sudo chown -R $USER:$USER /mnt/f/repos/trading_with_short
```

### Issue: MCP server connection errors

**Solution:** Ensure all dependencies are installed:
```bash
uv sync
```

Also verify that `npx` can run MCP servers:
```bash
npx -y @modelcontextprotocol/server-brave-search --version
```

---

## Running in Background (Optional)

### Using nohup

```bash
# Run trading floor in background
nohup uv run trading_floor.py > trading_floor.log 2>&1 &

# Run Gradio UI in background
nohup uv run app.py > app.log 2>&1 &
```

### Using systemd (Advanced)

You can create systemd service files to run the application as a service. This is more complex and typically not necessary for development.

---

## Stopping the Application

- If running in foreground: Press `Ctrl+C`
- If running in tmux: `Ctrl+B`, then `x` to kill the pane, or `Ctrl+C` in the pane
- If running with nohup: Find the process and kill it:
  ```bash
  ps aux | grep "trading_floor.py"
  kill <PID>
  ```

---

## Next Steps

1. Monitor the trading activity through the Gradio UI
2. Check logs in the terminal or log files
3. Adjust `.env` settings to customize behavior:
   - Change `RUN_EVERY_N_MINUTES` to run more/less frequently
   - Set `USE_MANY_MODELS=true` to use different LLMs for each trader
   - Set `RUN_EVEN_WHEN_MARKET_IS_CLOSED=true` to run 24/7

---

## Additional Notes

- The application uses SQLite for data storage (`accounts.db` file)
- Market data is cached in the database to reduce API calls
- The application creates memory databases for each trader in the `memory/` directory
- All logs are stored in the SQLite database

---

## Summary of Commands

```bash
# Navigate to project
cd /mnt/f/repos/trading_with_short

# Install dependencies
uv sync

# Run trading floor
uv run trading_floor.py

# Run Gradio UI (in separate terminal)
uv run app.py
```

---

For more information, refer to the `readMe.md` file in the project directory.
