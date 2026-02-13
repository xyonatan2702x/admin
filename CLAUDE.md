# CLAUDE.md - AI Assistant Guide for Admin Dashboard

## Project Overview

A personal productivity dashboard (Hebrew, RTL) deployed as a single-page app on GitHub Pages. Features task management, habit tracking with XP system, and cross-device sync via Firebase. Everything lives in a single `index.html` file (~1566 lines) with no build system.

## Architecture

### Single-File SPA

The entire application is in `index.html`. There is no build step, no bundler, no package manager. All dependencies are loaded from CDNs at runtime.

```
admin/
├── .nojekyll          # Disables Jekyll on GitHub Pages
├── index.html         # Entire application (HTML + CSS + JS/React)
└── CLAUDE.md          # This file
```

### Code Organization (inside index.html)

The file is organized in linear sections, marked with `// ===== SECTION =====` comments:

| Lines (approx) | Section |
|---|---|
| 1–149 | HTML head, CSS, CDN imports, Tailwind config, animations |
| 185–217 | Firebase configuration and initialization |
| 220–373 | Auth context, `useAuth()` hook, `useFirestoreSync()` hook |
| 375–426 | Utility functions and constants (PRIORITIES, CATEGORIES, NAV_ITEMS) |
| 427–543 | `SetupScreen` component (Firebase setup wizard) |
| 546–662 | `LoginScreen` component (email/password auth) |
| 665–685 | `SyncIndicator` component |
| 687–807 | `TaskItem` component |
| 810–920 | `TaskModal` component (add/edit tasks) |
| 921–1095 | `TasksPanel` component (main task management) |
| 1098–1323 | `HabitsPanel` component (habit tracker with XP) |
| 1326–1341 | `BotsPanel` component (placeholder) |
| 1345–1360 | `AnalyticsPanel` component (placeholder) |
| 1364–1416 | `SettingsPanel` component |
| 1419–1545 | `Dashboard` component (main layout, navigation) |
| 1548–1566 | `App` root component, React 18 `createRoot`, icon init |

## Tech Stack

- **UI**: React 18.3.1 + ReactDOM (CDN, Babel JSX transpilation in-browser)
- **Styling**: Tailwind CSS (CDN) + custom CSS animations
- **Icons**: Lucide 0.344.0 (CDN)
- **Backend**: Firebase 10.12.0 compat (Auth + Firestore)
- **Auth**: Email/Password via Firebase Authentication
- **Storage**: Firestore (cloud) + localStorage (local fallback)
- **Deployment**: GitHub Pages (static file hosting)
- **Language**: JavaScript (JSX via Babel standalone)

## Key Patterns

### State Management

- **React Context API** for auth state (`AuthContext` / `AuthProvider`)
- **`useFirestoreSync(localKey, firestoreCollection, defaultValue)`** — custom hook that:
  - Reads from localStorage on mount
  - Syncs bidirectionally with Firestore when authenticated
  - Merges local + remote on first load (remote wins if both exist)
  - Debounces writes to Firestore (800ms)
  - Listens for real-time updates from Firestore

### Data Models

**Task**:
```javascript
{
  id: string,              // generateId(): timestamp + random
  title: string,
  description: string,
  priority: 'urgent' | 'high' | 'medium' | 'low',
  category: string,        // work, personal, health, learning, finance, home, bots, other
  dueDate: string,         // ISO date string
  subtasks: [{ id, title, completed }],
  completed: boolean,
  createdAt: string         // ISO string
}
```

**Habit**:
```javascript
{
  id: number,
  name: string,
  xp: number,
  color: string,           // Tailwind color class
  category: string
}
```

**Habit completion data**: `{ [YYYY-MM-DD]: [habitId, ...] }`

### Firestore Structure

```
users/{uid}/data/tasks        → { value: Task[], updatedAt: serverTimestamp }
users/{uid}/data/habits       → { value: Habit[], updatedAt: serverTimestamp }
users/{uid}/data/habitData    → { value: {...}, updatedAt: serverTimestamp }
```

### localStorage Keys

- `dashboard_tasks` — task array
- `habits` — habit definitions
- `habitData` — daily completion records
- `dashboard_activePanel` — last active panel

## Development Workflow

### Making Changes

1. Edit `index.html` directly — there is no build step
2. Open in a browser to test (or use a local HTTP server)
3. Commit and push to deploy to GitHub Pages

### Testing

There is no test framework. Manual browser testing is the only method:
- Open `index.html` in a browser
- Test with and without Firebase configured
- Test on mobile viewport (responsive breakpoint at 768px)

### Deployment

Push to the main branch → GitHub Pages serves `index.html` automatically. The `.nojekyll` file ensures GitHub Pages doesn't process the file through Jekyll.

### Firebase Configuration

Firebase config lives at lines 191–198 in `index.html`. Values must be filled in manually. The app works in local-only mode (localStorage) when config values are empty strings.

## Code Conventions

### Naming

- **PascalCase** for React components: `TasksPanel`, `LoginScreen`, `HabitsPanel`
- **camelCase** for functions, variables, hooks: `useAuth`, `generateId`, `formatDate`
- **SCREAMING_SNAKE_CASE** for constants: `FIREBASE_CONFIG`, `PRIORITIES`, `CATEGORIES`

### React Patterns

- Functional components only (no class components)
- Hooks for all state and effects (`useState`, `useEffect`, `useCallback`, `useMemo`, `useRef`)
- Context API for global state (auth)
- Props drilling for component communication (no Redux/Zustand)

### UI/UX Conventions

- **Hebrew language** throughout — all UI text, labels, placeholders, error messages are in Hebrew
- **RTL layout** (`<html dir="rtl">`)
- **Mobile-first** responsive design with Tailwind
- **Inline Tailwind classes** for all styling (no separate CSS files beyond animations)
- Long `className` strings are normal in this codebase

### ID Generation

Uses `generateId()`: `Date.now().toString(36) + Math.random().toString(36).substr(2)`

## Important Notes for AI Assistants

1. **Single file**: All changes go in `index.html`. Do not create separate .js/.css files unless explicitly restructuring.
2. **No build system**: No npm, no bundler. CDN-only dependencies. Do not add package.json unless asked.
3. **Hebrew/RTL**: All user-facing text must be in Hebrew. Layout is right-to-left.
4. **Firebase is optional**: The app must work without Firebase (localStorage fallback). Always guard Firebase calls with `firebaseReady` checks.
5. **CDN dependencies**: When adding libraries, use CDN links (prefer cdnjs.cloudflare.com). Add `<script>` tags in the `<head>`.
6. **No secrets in repo**: Firebase config fields are empty strings by default. Never commit actual API keys.
7. **Panels**: Bots and Analytics panels are placeholders ("Coming Soon"). The active panels are Tasks, Habits, and Settings.
8. **Offline-first**: The app should always work offline via localStorage. Firestore sync is an enhancement, not a requirement.
