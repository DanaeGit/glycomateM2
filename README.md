# GlycoMate — Milestone 2 (Developer)

A small diabetes self-management prototype that links **food intake** and **physical activity** to a daily **energy balance**, then returns short, motivational insights.

- **Frontend**: React Native + Expo → `/mobile`
- **Backend**: Spring Boot → `/server`
- **Architecture**: MVVM (Views ↔ ViewModels ↔ Model/Repositories), thin REST API
- **Role (Milestone 2)**: Developer (focus on login, password reset, calorie calculation steps, and encouragement messages)

---

## 1. Project Scope (Milestone 2)

Implemented features:
- **Auth**: Login, password reset (email-style flow simulated by API status).
- **Calorie flow**: Intake → Activity → Result → Review (net energy & suggestion).
- **Insights**: Simple “nudge” sentences from the computed balance.
- **Basic health ping** for liveness.

Non-goals (deferred): photo-based meal recognition, full nutrition database, device auto-sync.

---

## 2. Repository Structure
```
├─ mobile/ # React Native (Expo)
│ ├─ App.tsx
│ ├─ app.json / app.config.ts
│ ├─ .env.example # copy to .env and set API_URL
│ └─ src/
│ ├─ api/ # API client + viewmodel hooks
│ │ ├─ client.ts # base axios/fetch client (reads API_URL)
│ │ ├─ auth.ts # login/reset requests
│ │ └─ calc.ts # intake/activity/compute requests
│ ├─ components/ # shared UI (Button, TextField, Card, etc.)
│ ├─ navigation/ # stack/tab navigator
│ └─ screens/ # Views
│ ├─ auth/
│ │ ├─ LoginScreen.tsx
│ │ └─ ResetPasswordScreen.tsx
│ ├─ calorie/
│ │ ├─ IntakeStepScreen.tsx
│ │ ├─ ActivityStepScreen.tsx
│ │ ├─ ResultScreen.tsx
│ │ └─ ReviewStepScreen.tsx
│ └─ insights/
│ └─ DailyInsightsScreen.tsx
│
├─ server/ # Spring Boot
│ ├─ pom.xml
│ └─ src/main/java/com/glycomate/
│ ├─ config/ # CORS, minimal security
│ ├─ domain/ # Entities/DTOs
│ │ ├─ AuthDtos.java # LoginRequest, ResetRequest, AuthResponse
│ │ ├─ CalcDtos.java # IntakeDto, ActivityDto, ComputeRequest, ComputeResponse
│ │ └─ InsightDtos.java # InsightDto
│ ├─ infra/ # Repositories/Services (in-memory demo)
│ │ ├─ AuthService.java
│ │ ├─ CalcService.java # core energy balance logic
│ │ └─ InsightService.java
│ ├─ view/ # Controllers (REST)
│ │ ├─ AuthController.java
│ │ ├─ CalcController.java
│ │ ├─ HealthController.java
│ │ └─ ReportController.java
│ └─ viewmodel/ # Response mappers / VMs
│
├─ docs/
└─ README.md
```
---

## 3. Architecture (MVVM in practice)

- **View (mobile/src/screens/…)**: renders UI, collects input, reacts to ViewModel state.  
- **ViewModel (mobile/src/api/… + page hooks)**: orchestrates validation, shapes requests/responses, holds transient state, calls API client.  
- **Model/Repositories (server/infra, server/domain)**: implements business logic (energy math), stores transient data (in-memory for demo), prepares DTOs.

Why MVVM:
- Keeps **UI** lean; puts **business/state** in ViewModel & services.
- React components bind to ViewModel state (props/hooks) → simpler testing & refactoring.

---

## 4. How to Run (Local Development)

### 4.1 Backend (Spring Boot)
**Prereqs**: JDK 17, Maven 3.9+
```powershell
cd server
mvn clean spring-boot:run
# => http://localhost:8080
```
Change port:
```powershell
mvn spring-boot:run -Dserver.port=8081
```
---
### 4.2 Mobile (Expo)

**Prereqs**: Node 18 LTS, npm 9/10, Expo Go (phone) or emulator
```powershell
cd mobile
npm install
copy .env.example .env
```
Edit `mobile`/.env:
```ini
#use your PC’s LAN IPv4 so a phone can reach the server
API_URL=http://192.168.xx.yy:8080
```
Run:
```powershell
npx expo start
```
- Press a (Android emulator) or scan QR in Expo Go.
- Phone and PC must be on the same Wi-Fi.

**Common pitfalls**
- Don’t use `localhost` for a phone; use PC LAN IP (`ipconfig`).
- If CORS/network errors → allow Java in Windows Firewall.

---

## 5. Program Logic (Core)
### 5.1 Energy balance (server/infra/CalcService.java)
- Inputs: intake (kcal), activity (steps/minutes → kcal).
- Output: `netKcal = intakeKcal – expenditureKcal` and advice:
  - `|netKcal| < 150` → “On track—nice job.”
  - `|netKcal| ≥ 150` → “Slight surplus—consider a short walk.”
  - `|netKcal| ≤ -150` → “Slight deficit—remember to refuel.”
Expenditure uses a lightweight estimate (e.g., steps × factor or minutes × MET × weight).
 For demo, weight/MET are constants.
---
### 5.2 Auth flow (server/view/AuthController.java)
- `POST /auth/login` → returns demo token.
- `POST /auth/reset` → returns `200` OK (simulated email reset).

---

### 5.3 Insights (server/view/ReportController.java)
Returns latest computed message for the Daily Insights screen.

---

## 6.Environment & Configuration
**Mobile** (`/mobile/.env`)
```ini
API_URL=http://<LAN-IP>:8080
```

Restart Expo after changes or `npx expo start -c`.
**Server** (`/server/src/main/resources/application.properties`)
- Port, logging level, CORS.
Run-time override: -Dserver.port=8081.

---

## 7. Quality, Testing & Lint
**Frontend**
- Functional focus: step flow correctness, validation, API error banners.
- Optional unit tests for helpers in `src/api/`* (`formatKcal`, `calcSuggestion`).
```bash
npm run lint
npm run fix
```
**Backend**
- Unit tests for `CalcService` (positive/boundary/error).
```bash
mvn clean package
```

## 8. Troubleshooting
- Network: Use LAN IP, confirm server alive in browser, allow Java through firewall.
- Metro cache: `npx expo start -c`
- CORS: check server CORS config.
- Port busy: change server port & update `API_URL`.

