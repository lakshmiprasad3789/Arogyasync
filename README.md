# Personal Health Records & AI Health Assistant

A web application prototype for organizing personal health information in one place. It provides a dashboard for health records, document analysis, medication tracking, laboratory trends, a health timeline, family/caregiver features, and FHIR-oriented data views. Gemini-powered server endpoints support document analysis, plain-language summaries, and a health assistant.

> **Important:** This is a software prototype, not a medical device or a substitute for a qualified healthcare professional. AI-extracted details may be incorrect or incomplete. Always verify medicines, dosages, diagnoses, dates, and test values against the original records and consult a clinician before making healthcare decisions.

## Features

- **Health dashboard:** Overview of health information and calculated health indicators.
- **Document library:** Review and manage uploaded or sample medical records.
- **AI document analysis:** Send document content to the backend for AI-assisted extraction.
- **Health summaries:** Generate readable summaries of health information.
- **Health assistant:** Ask questions about health records through a Gemini-powered endpoint.
- **Medication manager:** Track medication details, doses, and refill-related information.
- **Lab trends:** View laboratory results and trends over time.
- **Health timeline:** Review health events chronologically.
- **Family and caregiver tools:** Family network, caregiver-mode indicators, and SOS alert UI.
- **FHIR / ABDM view:** Mapping and presentation utilities for FHIR-style health data. This UI alone does not certify ABDM integration or compliance.
- **Privacy settings and language support:** Application preferences and localization utilities.

Feature availability depends on configuration and the current implementation. The app includes sample records for demonstration.

## Technology stack

- React 19 and TypeScript
- Vite 8 for frontend tooling
- Express 4 and `tsx` for the Node.js server
- Google Gemini via `@google/genai`
- Supabase JavaScript client (`@supabase/supabase-js`)
- Tailwind CSS 4, Lucide React, Motion, and Recharts

## Requirements

- Node.js version compatible with the installed Vite 8 toolchain
- npm (or Bun, since a `bun.lock` file is included)
- A Gemini API key for AI-powered endpoints
- Supabase project credentials if you intend to enable the server's Supabase integration

## Getting started

1. **Extract the project ZIP** and open a terminal in the project directory.
2. **Install dependencies:**

   ```bash
   npm install
   ```

   Alternatively, if you use Bun:

   ```bash
   bun install
   ```

3. **Configure environment variables.** Create a `.env` file in the project root. Start from `.env.example` and add the values relevant to your setup:

   ```env
   GEMINI_API_KEY=your_gemini_api_key
   APP_URL=http://localhost:3000

   # Optional: configure these only if using Supabase
   SUPABASE_URL=https://your-project.supabase.co
   SUPABASE_ANON_KEY=your_supabase_anon_key
   # Or use SUPABASE_PUBLISHABLE_KEY where appropriate
   ```

   The server also recognizes `SUPABASE_KEY` and `SUPABASE_SERVICE_ROLE_KEY`. **Never expose a Supabase service-role key in frontend code or commit it to source control.** Use the least-privileged key possible and configure database Row Level Security (RLS) policies before storing real patient data.

4. **Start the application:**

   ```bash
   npm run dev
   ```

   The package script starts the Express server with `tsx`. In development, the server mounts Vite middleware. The default port is `3000` unless `PORT` is set.

5. Open the local URL printed in the terminal (typically `http://localhost:3000`).

## Available scripts

| Command | Purpose |
| --- | --- |
| `npm run dev` | Start the server in development mode with Vite middleware |
| `npm run start` | Start the server using `tsx server.ts` |
| `npm run build` | Build the frontend with Vite |
| `npm run preview` | Run Vite's preview server for the built frontend |
| `npm run lint` | Run TypeScript checking with `tsc --noEmit` |
| `npm run clean` | Remove generated `dist` and `server.js` files (script uses a Unix-style command) |

Note: `npm run preview` previews the Vite build; it is not necessarily equivalent to running the Express API server in production. Verify the intended deployment setup before publishing.

## API endpoints

The Express server currently defines these routes:

| Method | Endpoint | Purpose |
| --- | --- | --- |
| `POST` | `/api/jobs/create` | Create an in-memory analysis job |
| `GET` | `/api/jobs/:jobId` | Check a job's status/result |
| `DELETE` | `/api/jobs/:jobId` | Delete a job |
| `POST` | `/api/analyze-document` | Run AI-assisted medical document analysis |
| `POST` | `/api/generate-summary` | Generate a health summary |
| `POST` | `/api/health-assistant` | Submit a prompt to the health assistant |

Request and response schemas are implemented in `server.ts`; inspect that file before integrating a separate client. Do not assume job records persist after a server restart unless persistence is added.

## Project structure

```text
.
├── app/
│   └── applet/                 # Applet metadata and HTML shell
├── src/
│   ├── components/
│   │   ├── abdm/                # FHIR / ABDM-oriented views
│   │   ├── assistant/           # AI health assistant UI
│   │   ├── auth/                # Login and account UI
│   │   ├── dashboard/           # Dashboard and health index cards
│   │   ├── documents/           # Document library and detail modal
│   │   ├── family/              # Family, caregiver, and SOS features
│   │   ├── labs/                # Lab trends
│   │   ├── layout/              # Navigation and layout
│   │   ├── medications/         # Medication and dose tracking
│   │   ├── privacy/             # Privacy settings
│   │   ├── summary/             # Health summary UI
│   │   ├── timeline/            # Health timeline
│   │   └── upload/              # Document upload flow
│   ├── context/                 # Shared health-record state
│   ├── data/                    # Demonstration/sample records
│   ├── types/                   # TypeScript data types
│   ├── utils/                   # Health calculations, FHIR mapping, i18n, etc.
│   ├── App.tsx                  # Main application views and modals
│   └── main.tsx                 # React entry point
├── server.ts                    # Express API and Gemini/Supabase setup
├── package.json
├── .env.example
└── vite.config.ts
```

## Data storage and Supabase

The current frontend context stores several items—such as account/profile data, selected records, medication dose logs, language preferences, and SOS alerts—in browser `localStorage`. This means data is browser/device-specific and should not be treated as a secure, synchronized medical-record database.

The server initializes a Supabase client when credentials are configured. Configure your Supabase schema, access policies, and persistence paths explicitly before relying on it for real data. Adding credentials alone does not prove that every frontend feature is saved to Supabase.

For any real patient data, implement authentication and authorization, RLS, secure file storage, retention/deletion rules, audit logging, and appropriate privacy/security reviews. Avoid using real patient information in demos or public repositories.

## AI and document extraction

The backend uses the Google GenAI SDK and reads `GEMINI_API_KEY` from the server environment. Configure the key in your local `.env` file or your hosting provider's secret manager. In Google AI Studio, use its Secrets configuration if that is how the app is deployed.

AI output is probabilistic. For medical documents—especially handwritten prescriptions—show the source document next to extracted fields and require user verification. Do not silently convert uncertain handwriting into medication instructions.

## FHIR / ABDM notes

The project includes FHIR mapping utilities and an ABDM/FHIR-oriented UI. FHIR is a healthcare data interoperability standard; JSON is a data format, not a synonym for FHIR. A working production integration typically requires validated FHIR resources, correct profiles and identifiers, consent and authorization flows, and integration with the relevant ABDM services. Do not describe the app as ABDM-certified solely because it has an FHIR view.

## Troubleshooting

- **AI endpoints report a missing API key:** Check that `GEMINI_API_KEY` is present in the server environment and restart the server.
- **Supabase is not initialized:** Verify `SUPABASE_URL` and one supported key variable. Check server logs for configuration messages.
- **Port already in use:** Set `PORT` to an available port before starting the app.
- **Changes are not reflected:** Restart the development server and refresh the browser.
- **TypeScript/build errors:** Run `npm run lint` and `npm run build`, then resolve the reported errors.
- **Local data disappeared:** Browser storage can be cleared, differs by browser/profile, and is not automatically synced across devices.

## Security checklist before deployment

- Keep API keys and privileged database keys server-side.
- Never commit `.env`; check `.gitignore` before pushing to GitHub.
- Configure Supabase RLS and test access as different users.
- Add appropriate authentication, authorization, validation, rate limiting, and error handling.
- Restrict and validate uploaded files and request sizes.
- Avoid logging sensitive medical information.
- Use HTTPS and review hosting, backups, deletion, and incident-response practices.
- Do not use this prototype for clinical decisions without qualified review and appropriate validation.

## License

The source entry point contains an Apache-2.0 SPDX notice. Confirm the intended license and add a complete `LICENSE` file before redistributing the project.
