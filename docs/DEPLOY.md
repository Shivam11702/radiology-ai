# Deploy Radiology AI

## Option A — Local (Docker, fastest for demo day)

```powershell
cd radiology-ai
.\launch.bat
```

- App: http://localhost:3000
- API docs: http://localhost:8000/docs
- Login: `doctor@radiology.ai` / `demo1234`

Requires Docker Desktop running.

---

## Option B — Cloud (public URL for judges)

### 1. Backend on Render

1. Go to [Render Dashboard](https://dashboard.render.com/) → **New** → **Blueprint**
2. Connect GitHub repo `Shivam11702/radiology-ai`
3. Render reads `render.yaml` and creates:
   - **radiology-ai-api** (FastAPI + PyTorch CPU Docker image)
   - **radiology-ai-redis** (job queue)
4. Set environment variables on Render:
   - `ENABLE_BASELINE_MODELS` = `1`
   - `GEMINI_API_KEY` = `your-gemini-key`
   - `GEMINI_MODEL` = `gemini-1.5-flash`
5. Wait for the API build (~10–15 min first time — PyTorch image is large)
6. Copy the API URL, e.g. `https://radiology-ai-api.onrender.com`

### 2. Frontend on Vercel

1. Go to [Vercel](https://vercel.com/new) → Import `Shivam11702/radiology-ai`
2. Set **Root Directory** to `frontend`
3. Add environment variable:
   - `VITE_API_URL` = `https://radiology-ai-api.onrender.com/api/v1`
4. Deploy

Your live app will be at `https://your-project.vercel.app`.

### 3. Update CORS (recommended)

In Render → **radiology-ai-api** → Environment:

```
CORS_ORIGINS=["https://your-project.vercel.app"]
```

Replace with your actual Vercel URL.

---

## Option C — Full stack on a VPS

```bash
git clone https://github.com/Shivam11702/radiology-ai.git
cd radiology-ai/docker
docker compose up -d --build
```

Open port 3000 (frontend) and 8000 (API) in your firewall.

---

## Notes

- First API request after idle may take 30–60s on Render free tier (cold start).
- Celery async jobs need Redis; sync **Analyze** flow works without the worker.
- Demo login: `doctor@radiology.ai` / `demo1234`
- Change `SECRET_KEY` in production (Render generates one via blueprint).
