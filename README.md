# RateMyProfBot
![bot preview](./PROF.png)
Automated RateMyProfessors submission CLI. Fills and submits professor ratings with AI-generated unique reviews using [OpenRouter](https://openrouter.ai) and/or [Groq](https://groq.com).

## Features
- Automated form filling for any professor by ID or URL
- AI-generated reviews via OpenRouter or Groq (unique per professor, stored in local history)
- Smart model rotation — cycles through all available models with 1hr cooldown on failure
- Sentiment-aware reviews — negative tone for low ratings, positive for high, mixed for middle
- Grade & tag randomization — auto-picks appropriate grades/tags based on rating
- Parallel workers — run multiple submissions simultaneously with true parallelism
- Loop mode — run batches continuously until a target count is hit or all cookies exhausted
- Cookie-based account mode — load saved sessions from a single JSON file
- Cookie generator (`cookiegen.js`) — auto-creates RMP accounts and saves cookies to file
- 1 review per cookie per professor — enforced via SHA-256 cookie identity hashing
- Mutex-safe cookie picking — parallel workers never grab the same account
- Duplicate detection — never submits the same review twice per professor
- Review validation — local + AI checks ensure clean output (no names, no colons, no quotes)
- Cookie banner handling — auto-dismisses and re-fills form if state resets

## Requirements
- Node.js 18+
- An [OpenRouter](https://openrouter.ai) API key and/or a [Groq](https://groq.com) API key

## Installation
```bash
git clone https://github.com/y8olol/RateMyProfBot.git
cd RateMyProfBot
npm install
npx playwright install chromium
```

## Usage
Grab the professor's numeric ID from their RateMyProfessors URL (the digits at the end).

### Basic interactive
```bash
node rmpcli.js --url USERID
```

### With AI review (OpenRouter)
```bash
node rmpcli.js --url USERID --rating 4 --difficulty 2 -k sk-or-...
```

### With AI review (Groq)
```bash
node rmpcli.js --url USERID --rating 1 --difficulty 5 -g gsk_...
```

### With both keys (auto-rotates, falls back on failure)
```bash
node rmpcli.js --url USERID --rating 1 -k sk-or-... -g gsk_... --headless
```

### Parallel workers + loop
```bash
# Run forever, 5 workers per batch (Ctrl+C to stop)
node rmpcli.js --url USERID --rating 1 -w 5 --loop -g gsk_... --headless

# Run until 50 total submitted
node rmpcli.js --url USERID --rating 1 -w 5 --loop --total 50 -g gsk_... --headless
```

### Cookie account mode
```bash
# Generate accounts first
node cookiegen.js -n 20 -H

# Run with cookie pool — stops automatically when all accounts used
node rmpcli.js --url USERID --rating 1 -w 5 --loop -g gsk_... --headless --cookies ./cookies.json
```

## Options

| Flag | Alias | Default | Description |
|------|-------|---------|-------------|
| `--url` | `-u` | — | Professor URL or numeric ID |
| `--rating` | | `5` | Overall rating (0–5) |
| `--difficulty` | | `2` | Difficulty (0–5) |
| `--wouldTakeAgain` | | `Yes` | Yes / No |
| `--forCredit` | | `Yes` | Yes / No |
| `--usesTextbooks` | | `No` | Yes / No |
| `--attendanceMandatory` | | `No` | Yes / No |
| `--grade` | | auto | Grade received (auto-picked based on rating if omitted) |
| `--courseCode` | | `111` | Course code |
| `--review` | | — | Manual review text (skips AI) |
| `--openrouterKey` | `-k` | — | OpenRouter API key |
| `--groqKey` | `-g` | — | Groq API key |
| `--cookies` | `-c` | — | Path to multi-account cookies JSON file |
| `--workers` | `-w` | `1` | Parallel workers per batch |
| `--loop` | `-l` | `false` | Loop in batches |
| `--total` | `-t` | `0` | Total submissions (0 = infinite) |
| `--headless` | `-H` | `false` | Run browser headlessly |
| `--slowMo` | | `300` | Milliseconds between actions |

## Cookie Generator

```bash
# Generate 10 accounts headlessly, saved to cookies.json
node cookiegen.js -n 10 -H

# Custom output file
node cookiegen.js -n 5 -o ./myaccounts.json
```

| Flag | Alias | Default | Description |
|------|-------|---------|-------------|
| `--count` | `-n` | `1` | Number of accounts to generate |
| `--output` | `-o` | `cookies.json` | Output file path |
| `--headless` | `-H` | `false` | Run browser headlessly |

## Cookies File Format
```json
[
  { "rmpAuth": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." },
  { "rmpAuth": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." }
]
```
Each entry is one account. The bot picks the first unused account per professor per run, tracked by a SHA-256 hash of the cookie value.

## AI Model Rotation
When multiple models are available, the bot cycles through them automatically. A model that returns an error is put on a **1 hour cooldown** before being tried again.

**Groq models (tried first):** `llama-3.3-70b-versatile`, `llama-3.1-8b-instant`, `moonshotai/kimi-k2-instruct`, `qwen/qwen3-32b`, `meta-llama/llama-4-scout-17b-16e-instruct`, `allam-2-7b`

**OpenRouter models (fallback):** `arcee-ai/trinity-large-preview:free`, `mistralai/mistral-7b-instruct:free`, `huggingfaceh4/zephyr-7b-beta:free`

## Review History
Submitted reviews are stored in `review_history/<profId>.json`. This file is gitignored. The bot will never generate a review that duplicates one already in the history for that professor.

## Notes
- `review_history/` and `cookies.json` are created automatically on first run
- Grades and tags are auto-randomized based on rating when not explicitly set
- All workers in a batch pre-generate their AI reviews in parallel before any browser launches

## Future Updates

- Custom user agents for workers
- Proxy support
- Crypto Miner
