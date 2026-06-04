# Deployment Guide — Circuitry

This guide covers deploying Circuitry as Option 1: Frontend on Vercel + Backend on Render.

---

## Step 1: Deploy Backend to Render

### 1.1 Create a Render Account
Go to [render.com](https://render.com) and sign up.

### 1.2 Connect GitHub Repository
1. In Render dashboard, click **New +** → **Web Service**
2. Select **Build and deploy from a Git repository**
3. Connect your GitHub account and select the `circuitry` repository

### 1.3 Configure the Service
- **Name**: `circuitry-backend`
- **Root Directory**: `server`
- **Runtime**: Node
- **Build Command**: `npm install`
- **Start Command**: `npm start`
- **Plan**: Free (or paid for reliability)

### 1.4 Add Environment Variables
In the Render dashboard under "Environment":
```
PORT=3001
CORS_ORIGIN=https://your-app.vercel.app
```

### 1.5 Deploy
Click **Create Web Service**. Render will build and deploy automatically. Once live, you'll get a URL like: `https://circuitry-backend.onrender.com`

---

## Step 2: Deploy Frontend to Vercel

### 2.1 Create a Vercel Account
Go to [vercel.com](https://vercel.com) and sign up.

### 2.2 Import Project
1. Click **Add New...** → **Project**
2. Select **Import Git Repository**
3. Connect GitHub and select the `circuitry` repository

### 2.3 Configure Project
- **Project Name**: `circuitry`
- **Framework Preset**: Vite
- **Root Directory**: `client`
- **Build Command**: `npm run build` (Vercel auto-detects from vercel.json)
- **Output Directory**: `dist`

### 2.4 Add Environment Variables
In Vercel dashboard under **Settings** → **Environment Variables**:
```
VITE_API_URL=https://circuitry-backend.onrender.com
```

### 2.5 Deploy
Click **Deploy**. Vercel will build and deploy automatically. Your frontend will be live at `https://circuitry.vercel.app` (or your custom domain).

---

## Step 3: Update Backend CORS

Once your Vercel URL is ready, update the backend's `CORS_ORIGIN` on Render:

1. Go to Render dashboard → your **circuitry-backend** service
2. Click **Environment** (or **Settings**)
3. Update `CORS_ORIGIN` to your Vercel URL: `https://your-app.vercel.app`
4. Click **Save** — the service will auto-redeploy

---

## Step 4: Test

1. Visit your Vercel frontend: `https://your-app.vercel.app`
2. Try generating a circuit — it should call your Render backend
3. Check browser console (F12) for any API errors

---

## Troubleshooting

### CORS Errors
- Ensure `CORS_ORIGIN` on Render matches your Vercel URL exactly
- On Render, the service must be restarted after env var changes

### Backend Not Responding
- Check Render logs: Dashboard → Service → **Logs**
- Ensure `VITE_API_URL` in Vercel matches your Render URL

### Build Failures
- **Frontend**: Check Vercel **Build Logs** in dashboard
- **Backend**: Check Render **Logs** in dashboard

---

## Advanced: Custom Domain

1. **Frontend**: In Vercel settings, add your domain (e.g., `circuitry.com`)
2. **Backend**: In Render settings, add a custom domain (paid plans only) or keep the `.onrender.com` subdomain
3. Update `CORS_ORIGIN` to your custom frontend domain

---

## Local Development

To test locally before deploying:

```bash
cd circuitry

# Terminal 1: Backend
cd server
npm install
npm run dev

# Terminal 2: Frontend
cd client
npm install
npm run dev
```

Visit `http://localhost:5173` (frontend) and it will call `http://localhost:3001` (backend).

---

## Summary

| Component | Platform | URL |
|-----------|----------|-----|
| Frontend | Vercel | `https://your-app.vercel.app` |
| Backend | Render | `https://circuitry-backend.onrender.com` |

Your Circuitry app is now live! 🚀
