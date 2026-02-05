# Moltworker Setup Progress

## Project Overview
Moltworker runs OpenClaw (AI assistant) on Cloudflare Workers using Cloudflare Sandbox (containers). This deployment is configured to use **Google Gemini via Cloudflare AI Gateway** with stored credentials.

---

## CURRENT STATUS: BLOCKED BY CLOUDFLARE ACCESS

### Pick Up Here Tomorrow

The worker is **deployed and running**, but a Cloudflare Access application is blocking access to the Control UI. You're seeing "Please contact your administrator to enable the Access App Launcher application".

**IMMEDIATE NEXT STEP:**

1. Go to [Zero Trust → Access → Applications](https://one.dash.cloudflare.com/)
2. Find the `moltbot-sandbox` application
3. **Delete it** or **Disable it** (three dots menu → Disable)
4. Then test the Control UI:
   ```
   https://moltbot-sandbox.matthew-9f1.workers.dev/?token=fff7904e2938693074edc3780596ffc752e25eb10af83486fb4600e64fcdf0c8
   ```
5. First load takes 1-2 minutes for container startup - wait and refresh

**After Control UI works:**
- Test chat functionality
- Verify Gemini models with `/model` command
- Optionally reconfigure Access to protect only `/_admin/` path (not the whole domain)

---

## All Access URLs

### Moltworker URLs
| Purpose | URL |
|---------|-----|
| **Control UI (Chat)** | https://moltbot-sandbox.matthew-9f1.workers.dev/?token=fff7904e2938693074edc3780596ffc752e25eb10af83486fb4600e64fcdf0c8 |
| **Admin UI** | https://moltbot-sandbox.matthew-9f1.workers.dev/_admin/ |
| **Worker Base URL** | https://moltbot-sandbox.matthew-9f1.workers.dev |

### Cloudflare Dashboard URLs
| Purpose | URL |
|---------|-----|
| **Workers & Pages** | https://dash.cloudflare.com/?to=/:account/workers-and-pages |
| **moltbot-sandbox Worker** | https://dash.cloudflare.com/?to=/:account/workers/services/view/moltbot-sandbox |
| **AI Gateway** | https://dash.cloudflare.com/?to=/:account/ai/ai-gateway |
| **R2 Storage** | https://dash.cloudflare.com/?to=/:account/r2 |
| **Zero Trust (Access)** | https://one.dash.cloudflare.com/ |
| **Access Applications** | https://one.dash.cloudflare.com/ → Access → Applications |
| **API Tokens** | https://dash.cloudflare.com/profile/api-tokens |

### Credentials
| Item | Value |
|------|-------|
| **Gateway Token** | `fff7904e2938693074edc3780596ffc752e25eb10af83486fb4600e64fcdf0c8` |
| **Account ID** | `99f15c4644993721087349b552c8cf0f` |
| **AI Gateway Name** | `shilts-ai-gateway` |
| **Zero Trust Team Domain** | `mfsagent.cloudflareaccess.com` |
| **Access AUD Tag** | `dbbe5632805f6fd7fe7e3ba1f93d949b1ed9e4587eb042e97b9cbe30ac0c6525` |

---

## Completed Steps

### 1. Repository Setup
- Cloned from `https://github.com/cloudflare/moltworker`
- Applied PR #138 (multi-provider AI Gateway support): `gh pr checkout 138`
- Installed dependencies: `npm install`
- **Current branch:** `feat/multi-provider-ai-gateway`

### 2. Custom Code Modifications

**Changes made to support AI Gateway with stored credentials:**

#### `start-moltbot.sh` (lines ~255-330)
Modified to configure all providers (Gemini, Anthropic, OpenAI) automatically when `AI_GATEWAY_BASE_URL` is set, without requiring local API keys. AI Gateway handles authentication via stored credentials configured in the Cloudflare dashboard.

Key changes:
- Removed conditional checks for `GEMINI_API_KEY`, `ANTHROPIC_API_KEY`, `OPENAI_API_KEY`
- All providers configured when AI Gateway URL is present
- Updated Gemini models to: `gemini-2.5-flash`, `gemini-2.5-pro`, `gemini-2.0-flash`
- Default primary model: `google-ai-studio/gemini-2.5-flash`
- Fallback chain: Gemini 2.5 Pro → Haiku → Sonnet

#### `README.md`
- Fixed documentation mismatch: `GOOGLE_API_KEY` → `GEMINI_API_KEY`
- Updated to reflect stored credentials approach

### 3. Secrets Configured (all set via `wrangler secret put`)

| Secret | Value | Status |
|--------|-------|--------|
| `AI_GATEWAY_BASE_URL` | `https://gateway.ai.cloudflare.com/v1/99f15c4644993721087349b552c8cf0f/shilts-ai-gateway/compat` | ✅ Set |
| `AI_GATEWAY_API_KEY` | `NJYmmpDeO53qQmoGs_D2ylDlLFyMDTG-L7YBv1nm` | ✅ Set |
| `MOLTBOT_GATEWAY_TOKEN` | `fff7904e2938693074edc3780596ffc752e25eb10af83486fb4600e64fcdf0c8` | ✅ Set |
| `CF_ACCESS_TEAM_DOMAIN` | `mfsagent.cloudflareaccess.com` | ✅ Set |
| `CF_ACCESS_AUD` | `dbbe5632805f6fd7fe7e3ba1f93d949b1ed9e4587eb042e97b9cbe30ac0c6525` | ✅ Set |

### 4. Cloudflare Dashboard Setup
- **Workers Paid Plan:** ✅ Enabled
- **R2 Storage:** ✅ Enabled (bucket: `moltbot-data`)
- **AI Gateway:** ✅ `shilts-ai-gateway` configured
  - Provider: Google AI Studio (API key stored in dashboard)
  - Authenticated Gateway: Enabled
  - Endpoint: `/compat` (OpenAI-compatible)
- **Cloudflare Access:** ⚠️ Application created but blocking Control UI - needs to be disabled/deleted

### 5. Deployment
- **Status:** ✅ Successfully deployed
- **Docker:** Docker Desktop (required - Colima/Podman had registry issues)
- **Deploy command:** `npm run deploy`

### 6. Local Environment
- **Wrangler:** Logged in as `matthew@shilts.net`
- **Docker Desktop:** Installed and working
- **Working directory:** `/Users/matthew/ws/moltworker`

---

## Still To Do

### 1. Fix Cloudflare Access (IMMEDIATE)
The Access application is blocking the Control UI. Either:
- Delete/disable the Access application entirely, OR
- Reconfigure it to only protect `/_admin/` path

### 2. Test the Deployment
- Access Control UI (wait 1-2 min for container startup)
- Verify Gemini models appear with `/model` command
- Test chat functionality

### 3. R2 Persistence (Optional but Recommended)
For data persistence across container restarts:

```bash
wrangler secret put R2_ACCESS_KEY_ID
wrangler secret put R2_SECRET_ACCESS_KEY
wrangler secret put CF_ACCOUNT_ID
# CF_ACCOUNT_ID = 99f15c4644993721087349b552c8cf0f
```

Create R2 API token in dashboard: R2 → Overview → Manage R2 API Tokens → Create with Object Read & Write for `moltbot-data` bucket.

### 4. Reconfigure Access Properly (Optional)
If you want Access protection for admin UI only:
1. Create new Access application
2. Set domain to `moltbot-sandbox.matthew-9f1.workers.dev`
3. Set path to `/_admin/*` (only protect admin routes)
4. Add policy allowing `matthew@shilts.net`

### 5. Optional: Chat Integrations
```bash
wrangler secret put TELEGRAM_BOT_TOKEN
wrangler secret put DISCORD_BOT_TOKEN
wrangler secret put SLACK_BOT_TOKEN
wrangler secret put SLACK_APP_TOKEN
```

---

## Architecture

```
User → Control UI → Cloudflare Worker → Sandbox Container (OpenClaw)
                            ↓
                    AI Gateway (/compat endpoint)
                            ↓
                    Google AI Studio (Gemini)
```

---

## Useful Commands

```bash
# Deploy changes
npm run deploy

# Check deployment status
wrangler deployments list

# View live logs
wrangler tail

# Set a secret
wrangler secret put SECRET_NAME

# List secrets
wrangler secret list

# Check git changes
git diff
git status
```

---

## Troubleshooting

### "Access App Launcher" error
Cloudflare Access is blocking the worker. Go to Zero Trust → Access → Applications and delete/disable the moltbot-sandbox application.

### Container slow to start
First request takes 1-2 minutes. Wait and refresh.

### Models not appearing
Check AI Gateway configuration. Ensure Google AI Studio provider has valid API key.

### Docker issues during deployment
Use Docker Desktop (not Colima/Podman).

---

## Git Status

**Branch:** `feat/multi-provider-ai-gateway` (from PR #138)

**Modified files (not committed):**
- `README.md` - Documentation fixes
- `start-moltbot.sh` - Stored credentials support
- `package-lock.json` - From npm install
- `CLAUDE.md` - This file

To see changes: `git diff`

To commit (if desired):
```bash
git add -A
git commit -m "Add AI Gateway stored credentials support for Gemini"
```
