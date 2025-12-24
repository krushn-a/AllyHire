## AI Interview Assistant

AI-first interview simulator that preloads questions from a resume, speaks them aloud, records answers, batch-transcribes and scores them, then surfaces a dashboard for reviewers.

### Highlights
- Guided interview flow with TTS, pre-speak countdown, timers per difficulty, waveform viz, and auto-advance on timeout.
- Batch upload of all responses: uploads once, sends to AssemblyAI for STT, then scores with Gemini and returns feedback.
- Interviewer dashboard with search, sort, filters, modal details, scores, and per-question feedback.
- Resume parsing (PDF/DOCX) with fallback to manual info capture when fields are missing.
- State persisted locally via Redux Persist so refreshes do not drop progress.

### Tech Stack
- Frontend: React + Vite, Ant Design, Tailwind (v4), Redux Toolkit, Redux Persist, axios, lucide icons.
- Backend: Node.js (Express), AssemblyAI (STT), Google Gemini (Generative + scoring), multer for uploads, pdf-parse/mammoth for resume text.

### Project Layout
- [backend/server.js](backend/server.js) — Express app + route mounting.
- [backend/controllers/interviewController.js](backend/controllers/interviewController.js) — resume extract, question gen, summary gen, batch evaluation.
- [backend/controllers/transcriptionController.js](backend/controllers/transcriptionController.js) — single-file transcription endpoint.
- [backend/routes/interview.js](backend/routes/interview.js) and [backend/routes/transcribe.js](backend/routes/transcribe.js) — route wiring.
- [backend/services/ai.js](backend/services/ai.js) — Gemini model factory.
- [backend/services/resume.js](backend/services/resume.js) — PDF/DOCX text extraction.
- [backend/utils/aiJson.js](backend/utils/aiJson.js) — safe JSON parsing from LLM responses.
- [frontend/src/App.jsx](frontend/src/App.jsx) — tabbed layout for interviewee and dashboard.
- [frontend/src/components/IntervieweeTab.jsx](frontend/src/components/IntervieweeTab.jsx) — resume upload, info capture, question preload, interview start.
- [frontend/src/components/ChatInterface.jsx](frontend/src/components/ChatInterface.jsx) — TTS + countdown + recording + batch submission flow.
- [frontend/src/components/InterviewerTab.jsx](frontend/src/components/InterviewerTab.jsx) — candidate list, filters, detail modal with feedback.
- [frontend/src/services/api.js](frontend/src/services/api.js) — client API wrapper.

### Environment
Create two `.env` files at the project root folders.

Backend `.env`
- PORT=5000
- GEMINI_API_KEY=your_key
- GEMINI_MODEL=gemini-2.5-flash
- ASSEMBLYAI_API_KEY=your_key

Frontend `.env`
- VITE_API_BASE_URL=http://localhost:5000/api

### Quick Start
1) Backend
- cd backend
- npm install
- npm start

2) Frontend (new terminal)
- cd frontend
- npm install
- npm run dev

Open the Vite URL (default http://localhost:5173). The frontend expects the backend on http://localhost:5000.

### API Surface (backend)
- POST /api/extract-resume — multipart `resume` (pdf/docx); returns parsed fields + raw text.
- POST /api/generate-questions — body `{ candidateInfo }`; returns 6 questions (2 Easy, 2 Medium, 2 Hard).
- POST /api/generate-summary — body `{ candidateInfo, questions, answers, scores }`; returns short summary + average.
- POST /api/batch-evaluate — multipart `payload` JSON + `audios[]` (webm). Transcribes via AssemblyAI, scores via Gemini, returns transcripts + evaluations.
- POST /api/transcribe-audio — multipart `audio`; single-file AssemblyAI transcription.

### Frontend Flow (interviewee)
- Upload resume → auto parse; missing fields fall back to a quick form.
- Request mic access, preload 6 questions, then start.
- For each question: 3s pre-speak countdown → TTS playback → 5s pre-record countdown → recording starts with live waveform and per-difficulty timer (20/60/120s).
- After Q6, all blobs upload once, transcribe, score, and summarize; state is persisted locally.

### Frontend Flow (interviewer)
- Dashboard lists candidates with status, score, date; search by name/email; filter by status; sort by score/name/date.
- Detail modal shows contact info, per-question scores, transcripts, and Gemini feedback.

### Notes & Decisions
- Audio is kept in-memory and uploaded only once to limit API calls and avoid partial failures.
- Gemini prompts enforce JSON-only responses; [backend/utils/aiJson.js](backend/utils/aiJson.js) strips code fences before parsing.
- Difficulty timers live in [backend/constants/difficulty.js](backend/constants/difficulty.js) and mirror the frontend timer setup for consistency.
- Tailwind v4 is brought in via the CSS `@import 'tailwindcss';` approach.

### Potential Next Steps
- Add retries/backoff around LLM calls and AssemblyAI polling.
- Persist recordings to object storage for audit/replay.
- Add auth for interviewer dashboard.
- Add CI lint/test, plus rate limits on generation endpoints.