# Timesheet Tracker

> A free, AI-powered employee attendance system built for small businesses that can't afford enterprise HR software.

![Made with Firebase](https://img.shields.io/badge/Firebase-Firestore-orange?style=flat-square&logo=firebase)
![Uses Groq AI](https://img.shields.io/badge/AI-Groq%20Llama%203-blue?style=flat-square)
![face-api.js](https://img.shields.io/badge/ML-face--api.js-green?style=flat-square)
![Bootstrap 5](https://img.shields.io/badge/UI-Bootstrap%205-purple?style=flat-square)
![Deployed on Cloudflare](https://img.shields.io/badge/Deployed-Cloudflare%20Pages-orange?style=flat-square&logo=cloudflare)

---

## Why I built this

My parents run small businesses and tracking employee attendance was a constant headache — manual timesheets, forgotten clock-ins, and no way to verify if someone actually showed up. Enterprise HR software like Workday or SAP costs hundreds of dollars a month, which makes it completely inaccessible to small family businesses and neighbourhood establishments.

I built Timesheet Tracker as a free, production-ready alternative with AI-powered fraud detection that any small business can use.

---

## Features

### Employee
- Clock in and out via **photo selfie** or **GPS location** (or both)
- AI-powered **liveness detection** — detects if a real person is present (not a printed photo)
- **Face verification** — confirms the same person who clocked in is clocking out
- **Break tracking** — start/end breaks, automatically deducted from total hours
- Personal shift history with status badges
- Manual time correction requests
- Real-time shift progress bar

### Fraud Detection
- **GPS spoof detection** — compares device GPS against IP geolocation
- **Face descriptor matching** — catches buddy punching (someone clocking out for a colleague)
- **Liveness check** — random head movement prompt to prevent photo spoofing
- **Time-based flagging** — early/late clock-outs flagged and sent to admin instantly
- **Groq AI signal aggregation** — combines all fraud signals into a risk verdict (low/medium/high)

### Admin Panel
- Live dashboard — who's in, who's out, who's flagged
- Real-time alerts with bell notification for flagged entries
- **EmailJS integration** — automatic email to admin when a flagged entry occurs
- Full timesheet records with filters (date, status)
- Correction request approval/rejection
- **AI Weekly Summary** — Groq generates plain-English HR reports per employee
- **Punctuality Leaderboard** — ranks employees by on-time rate
- **Analytics charts** — hours worked, attendance rate, status breakdown (Chart.js)
- Interactive geofence map (Leaflet.js) with draggable pin and radius visualisation
- CSV export for payroll
- Dark/Light mode toggle
- Configurable work hours, late threshold, allowed radius

---

## AI & ML Stack

| Feature | Technology | How it works |
|---|---|---|
| Face detection | face-api.js (TinyFaceDetector) | Neural network runs locally in browser |
| Liveness check | face-api.js (FaceLandmark68Net) | Tracks 68 facial landmarks, detects nose movement |
| Face matching | face-api.js (FaceRecognitionNet) | Compares 128-number face descriptors, Euclidean distance < 0.6 = same person |
| GPS spoof detection | ip-api.com + Haversine formula | Compares device GPS vs IP geolocation, flags if >50km apart |
| Risk scoring | Groq (Llama 3.1 8B) | Aggregates all fraud signals into low/medium/high verdict |
| Weekly summary | Groq (Llama 3.1 8B) | Generates plain-English HR report from shift data |
| Late prediction | Groq + statistical analysis | Mean + standard deviation of past clock-ins to predict lateness |

> All face detection runs **locally in the browser** — photos are never sent to an external server for ML processing.

---

## Project Structure

```
timesheet-tracker/
├── index.html        # Login & registration page
├── employee.html     # Employee dashboard (clock in/out, history, corrections)
├── admin.html        # Admin panel (dashboard, alerts, analytics, settings)
└── README.md
```

---

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | Pure HTML, CSS, Bootstrap 5 |
| Database | Firebase Firestore |
| Authentication | Firebase Auth (Email/Password) |
| File storage | Base64 in Firestore (no Storage bucket needed) |
| Face ML | @vladmandic/face-api (browser-side) |
| Maps | Leaflet.js + OpenStreetMap (free, no API key) |
| AI/LLM | Groq API (Llama 3.1 8B Instant) |
| Email alerts | EmailJS |
| Charts | Chart.js |
| Hosting | Cloudflare Pages |

---

## Getting Started

### 1. Clone the repo
```bash
git clone https://github.com/YOUR_USERNAME/timesheet-tracker.git
cd timesheet-tracker
```

### 2. Set up Firebase
1. Go to [firebase.google.com](https://firebase.google.com) and create a project
2. Enable **Email/Password** authentication
3. Create a **Firestore** database (test mode, region: asia-southeast1)
4. Register a web app and copy the config

### 3. Set up Groq
1. Go to [console.groq.com](https://console.groq.com)
2. Create a free API key

### 4. Set up EmailJS
1. Go to [emailjs.com](https://emailjs.com) and create a free account
2. Connect your Gmail as an email service
3. Create an email template with variables: `{{employee_name}}`, `{{department}}`, `{{date}}`, `{{time}}`, `{{flag_type}}`, `{{risk_level}}`, `{{reason}}`, `{{admin_url}}`

### 5. Add your credentials
Replace the placeholder values in each HTML file:

```js
const firebaseConfig = {
  apiKey: "YOUR_API_KEY",
  authDomain: "YOUR_AUTH_DOMAIN",
  projectId: "YOUR_PROJECT_ID",
  storageBucket: "YOUR_STORAGE_BUCKET",
  messagingSenderId: "YOUR_SENDER_ID",
  appId: "YOUR_APP_ID"
};

const GROQ_KEY = "YOUR_GROQ_KEY";

const EMAILJS_SERVICE = 'YOUR_SERVICE_ID';
const EMAILJS_TEMPLATE = 'YOUR_TEMPLATE_ID';
const EMAILJS_KEY = 'YOUR_PUBLIC_KEY';
emailjs.init('YOUR_PUBLIC_KEY');
```

### 6. Set admin credentials
In `index.html` and `admin.html`, update:
```js
const ADMIN_EMAIL = "your-admin@email.com";
const ADMIN_PASSWORD = "YourSecurePassword";
```

### 7. Deploy
Drag and drop the folder into [Cloudflare Pages](https://pages.cloudflare.com) or connect your GitHub repo for automatic deployments.

---

## Tracking Modes

Employees choose their tracking mode when registering:

| Mode | Description |
|---|---|
| Photo | Takes a selfie to clock in/out. AI verifies face and liveness. |
| Location | GPS checked against workplace coordinates. Must be within set radius. |
| Both | Requires both a selfie and valid GPS location. Most secure. |

---

## Flag System

| Situation | Status | Action |
|---|---|---|
| Clock-in 30+ min late | Late | Flagged in record |
| Clock-out 0-30 min early | Suspicious | Flagged in record |
| Clock-out 30+ min early | Flagged | Instant admin alert + email |
| Clock-out 15-45 min late | Suspicious | Flagged in record |
| Clock-out 45+ min late | Flagged | Instant admin alert + email |
| Face mismatch at clock-out | Flagged | Instant admin alert + email |
| GPS vs IP mismatch >50km | Flagged | Instant admin alert + email |

---

## Security Notes

- Admin credentials are hardcoded — suitable for trusted internal use only
- All face detection runs client-side (no photos sent to external ML APIs)
- Firestore security rules restrict reads/writes to authenticated users only
- GPS spoof detection uses a 50km threshold to account for IP geolocation inaccuracy in dense cities like Singapore

---

## What I learned building this

- **Firebase Auth + Firestore** — real-time database design, security rules, and auth flows
- **Browser-side ML** — running neural networks locally with face-api.js without any server
- **LLM integration** — prompt engineering for structured JSON output and natural language summarisation
- **Fraud detection systems** — layering multiple signals (face, GPS, time, liveness) to make better decisions than any single signal alone
- **Production thinking** — building for non-technical users who need things to just work

---

## Author

**Zander** — Secondary 1, Express Stream, ACSI Singapore
Competitive programmer (NOI 2026 Preliminaries, UKCC 2025 Top Scorer)
CS50x completed (Primary 6)
Applying for Sentinel Cybersecurity Programme

---

## License

MIT License — free to use, modify, and deploy.