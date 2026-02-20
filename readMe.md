## Trading with shorting

This is based on the trading example in the Ed Donner's course https://github.com/ed-donner/agents/tree/main/6_mcp/community_contributions/trading_with_short. It is a demonstration of 'paper' trading of equities with agents and MCP tools. It is not meant for live trading.

Create a .env file in the project root with your API keys:
Add the following environment variables (replace with your actual API keys):

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

There are 4 traders with different trading strategies and possibly using different LLMs.
Its behaviour is influenced by these .env values

RUN_EVERY_N_MINUTES = int(os.getenv("RUN_EVERY_N_MINUTES", "60"))

RUN_EVEN_WHEN_MARKET_IS_CLOSED = (
    os.getenv("RUN_EVEN_WHEN_MARKET_IS_CLOSED", "false").strip().lower() == "true"
)

RESET_TRADERS = os.getenv("RESET_TRADERS", "true").strip().lower() == "true"

when true the 4 traders are reset to their starting values.

USE_MANY_MODELS = os.getenv("USE_MANY_MODELS", "false").strip().lower() == "true"

when true these models will be used, otherwise they all use gpt-4.1-mini
    model_names = [
        "gpt-4.1-mini",
        "deepseek-chat",
        "gemini-2.5-flash",
        "grok-3-mini-beta",
    ]


Start a new Terminal and first start gradio interface with command
uv run app.py 
Then open another terminal window and start the server using command
uv run trading_floor.py