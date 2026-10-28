# IFA SkillQuest architecture

IFA SkillQuest separates the candidate experience from the hiring operations
surface while sharing one assessment API and MongoDB data layer.

## Platform architecture

```mermaid
flowchart LR
    Candidate["Applicant browser"] --> Web["React + TypeScript SPA"]
    Recruiter["Admin browser"] --> Web
    Web -->|HTTPS JSON| API["Express API"]
    Web -->|Google OAuth| Google["Google Identity"]
    API --> Auth["Auth + role middleware"]
    API --> Assessment["Assessment controllers"]
    API --> Content["Question/game content"]
    Auth --> DB[("MongoDB Atlas")]
    Assessment --> DB
    Content --> DB
```

## Candidate assessment flow

```mermaid
flowchart TD
    Start["Choose Applicant"] --> SignIn["Sign in or register"]
    SignIn --> Profile["Complete candidate profile"]
    Profile --> Unblock["Unblock Me"]
    Unblock --> Minesweeper["Minesweeper"]
    Minesweeper --> Water["Water Capacity"]
    Water --> Questions["Question Game"]
    Questions --> Results["Results and ranking"]
    Unblock -. "tab-switch guard + autosave" .-> Guard["Assessment safeguards"]
    Minesweeper -. "tab-switch guard + autosave" .-> Guard
    Water -. "tab-switch guard + autosave" .-> Guard
    Questions -. "tab-switch guard + autosave" .-> Guard
```

## Deployment topology

```mermaid
flowchart LR
    GitHub["GitHub: main"] --> Frontend["Vercel project: frontend/\nVite build → dist"]
    GitHub --> Backend["Vercel project: backend/\nExpress serverless function"]
    Frontend -->|VITE_API_URL| Backend
    Backend -->|MONGODB_URI| Atlas[("MongoDB Atlas")]
    Google["Google OAuth origins"] -.-> Frontend
```

## Design principles

- **Signal over spectacle:** the interface uses clear progress, timing, and
  score states so candidates know what matters.
- **One visual language:** indigo identifies primary actions, cyan identifies
  progress and system health, and amber marks attention or milestones.
- **Separation of concerns:** React owns interaction and presentation, Express
  owns validation and persistence, and MongoDB remains the source of truth.
- **Safe configuration:** secrets stay in Vercel environment variables and are
  never committed to the repository.
