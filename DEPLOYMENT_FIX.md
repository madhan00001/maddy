# Fixing Maddy AI 404 after Vercel deployment

The frontend previously called relative `/api/...` URLs. This only works when the Express backend is served from the same origin. If Vercel hosts only the Vite frontend, those routes return 404.

## Recommended setup: Vercel frontend + Render backend

1. Push this project to GitHub.
2. Deploy the project as a Render Web Service. Use build command `npm install && npm run build` and start command `npm run start`. Add the backend environment variables from `.env.example`, including `GEMINI_API_KEY`, `JWT_SECRET`, and `FRONTEND_URL`. Set `FRONTEND_URL` to your exact Vercel origin, for example `https://your-app.vercel.app` (no trailing slash).
3. In Vercel → Project → Settings → Environment Variables, add `VITE_API_BASE_URL` with the Render service origin, for example `https://your-api.onrender.com` (no trailing slash). Do not add `/api` to the end.
4. Redeploy Vercel after adding the variable. Vite environment variables are baked into the frontend build.
5. Test `https://YOUR-RENDER-SERVICE.onrender.com/api/health`. It should return JSON with `status: "ok"`.
6. Open the Vercel app and send a message. In browser DevTools → Network, requests should go to `https://YOUR-RENDER-SERVICE.onrender.com/api/chat`, not to the Vercel domain and not to `localhost`.

## Important notes

- Never put `GEMINI_API_KEY` in a `VITE_` variable; VITE variables are exposed to browser code. Keep the Gemini key only on the backend.
- For Vercel preview deployments, add their exact origin to `FRONTEND_URL` if you need cross-origin requests from previews. Multiple origins can be comma-separated.
- The app uses SQLite and a local `vector_db` directory. These are not reliably persistent on a typical serverless filesystem and may be lost on redeploy/restart. Use persistent storage or a managed database/vector store for production data.
- If `/api/health` itself returns 404 on Render, the backend deployment/start command is not running this Express server. Check Render logs.
