# Deployment Guide

Step-by-step instructions for pushing to GitHub and deploying to Render + Vercel.

---

## Step 1 — Push to GitHub

Open your terminal, navigate to the project folder, and run these commands one by one:

```bash
# Go into the project root
cd "banking-rag-chatbot"

# Initialize git (skip if already done)
git init

# Stage all files (the .gitignore keeps secrets + venv out)
git add .

# First commit
git commit -m "feat: GenAI banking RAG chatbot - initial release"
```

Now create a new repository on GitHub:
1. Go to [github.com/new](https://github.com/new)
2. Name it `banking-rag-chatbot`
3. Set it to **Public** (required for free Render/Vercel)
4. Do NOT initialize with README (you already have one)
5. Click **Create repository**

Then connect and push:

```bash
git remote add origin https://github.com/YOUR_USERNAME/banking-rag-chatbot.git
git branch -M main
git push -u origin main
```

---

## Step 2 — Deploy Backend to Render

1. Go to [render.com](https://render.com) and sign in (free account)
2. Click **New +** → **Web Service**
3. Connect your GitHub account, then select `banking-rag-chatbot`
4. Fill in the settings:

| Setting | Value |
|---|---|
| **Name** | `banking-rag-chatbot-api` |
| **Region** | Oregon (US West) |
| **Branch** | `main` |
| **Root Directory** | `backend` |
| **Runtime** | Python 3 |
| **Build Command** | `pip install --upgrade pip && pip install -r requirements.txt` |
| **Start Command** | `uvicorn main:app --host 0.0.0.0 --port $PORT` |
| **Plan** | Free |

5. Click **Advanced** → **Add Environment Variable** and add these:

| Key | Value |
|---|---|
| `GEMINI_API_KEY` | your Gemini API key |
| `GROQ_API_KEY` | your Groq API key |
| `CHROMA_MODE` | `local` |
| `CHROMA_PERSIST_DIR` | `./data/chroma_db` |
| `EMBEDDING_MODEL` | `sentence-transformers/all-MiniLM-L6-v2` |
| `CHUNK_SIZE` | `500` |
| `CHUNK_OVERLAP` | `100` |
| `TOP_K_RESULTS` | `5` |
| `ALLOWED_ORIGINS` | `https://your-vercel-app.vercel.app` ← update after step 3 |
| `LOG_LEVEL` | `INFO` |

6. Click **Create Web Service**
7. Wait ~5 minutes for the first build (it downloads PyTorch + sentence-transformers)
8. Once green, copy your Render URL: `https://banking-rag-chatbot-api.onrender.com`

Test it:
```bash
curl https://banking-rag-chatbot-api.onrender.com/health
```

> **Important:** Free Render services spin down after 15 minutes of inactivity. The first request after sleep takes ~30 seconds. This is normal.

> **Note on ChromaDB data:** Free Render tier uses ephemeral storage. Your vector DB resets on each redeploy. Re-upload your documents after deploying. For permanent storage, you would need a paid Render plan with a Disk, or migrate to a hosted vector DB like Pinecone.

---

## Step 3 — Deploy Frontend to Vercel

1. Go to [vercel.com](https://vercel.com) and sign in with GitHub (free account)
2. Click **Add New** → **Project**
3. Import your `banking-rag-chatbot` repository
4. Set **Root Directory** to `frontend`
5. Add **Environment Variables**:

| Key | Value |
|---|---|
| `NEXT_PUBLIC_API_URL` | `https://banking-rag-chatbot-api.onrender.com` |

6. Click **Deploy**
7. Once done, copy your Vercel URL: `https://banking-rag-chatbot-xxx.vercel.app`

---

## Step 4 — Update CORS on Render

Now that you have your Vercel URL, go back to the Render dashboard:
1. Open your Web Service → **Environment**
2. Update `ALLOWED_ORIGINS` to `https://banking-rag-chatbot-xxx.vercel.app`
3. Click **Save Changes** (Render will redeploy automatically)

---

## Step 5 — Verify End-to-End

1. Open your Vercel URL in the browser
2. Upload `sample-banking-faq.txt` from the sidebar
3. Ask: *"What documents do I need to open a savings account?"*
4. You should receive a grounded answer with source citations

Your chatbot is live! 🎉

---

## Updating the App

After making code changes:

```bash
git add .
git commit -m "fix: your change description"
git push
```

Render and Vercel both auto-deploy on every push to `main`.
