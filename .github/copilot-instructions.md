# iNotebook React + Express AI Assistant Guidelines

## Architecture Overview

**iNotebook** is a full-stack notes app with clear separation:
- **Frontend**: React 17 with Context API (no Redux) for state management, React Router for navigation
- **Backend**: Express.js with MongoDB (Mongoose) running on port 5000
- **Authentication**: JWT-based with hardcoded secret in `backend/middleware/fetchuser.js`

### Key Data Flow
1. Frontend components consume `NoteContext` via `useContext(noteContext)`
2. Context methods (`getNotes`, `addNote`, `editNote`, `deleteNote`) make REST API calls to backend
3. Backend validates input via `express-validator`, verifies JWT via `fetchuser` middleware, then interacts with MongoDB

## Project Structure & Patterns

### Frontend (React)
- **Components** are functional React components using hooks
- **State management**: Centralized in `src/context/notes/NoteState.js` - single provider wrapping entire app in `App.js`
- **Components** (`Home.js`, `Notes.js`, `AddNote.js`, etc.) use `useContext(noteContext)` to access and update notes
- **Authentication**: Login/Signup components exist but auth tokens are currently hardcoded in `NoteState.js` (lines 12, 34, 51, 68)
- **Routing**: React Router v5 with routes: `/`, `/about`, `/login`, `/signup`

### Backend (Express + MongoDB)
- **Port**: 5000 with CORS enabled
- **Routes**: 
  - `/api/auth` - User registration and login (see `backend/routes/auth.js`)
  - `/api/notes` - CRUD operations for notes (see `backend/routes/notes.js`)
- **Models**: `User` (name, email, password, date) and `Note` (user ref, title, description, tag, date)
- **Middleware**: `fetchuser` extracts JWT from `auth-token` header and validates it
- **Database**: MongoDB on localhost:27017 via Mongoose

## Critical Developer Workflows

### Running Both Frontend & Backend
```bash
npm run both
```
This uses `concurrently` to run `react-scripts start` and `nodemon backend/index.js` simultaneously.

### Development
- **Frontend only**: `npm start` (runs on :3000)
- **Backend only**: `nodemon backend/index.js` (manual, port 5000)
- **Database**: Requires local MongoDB running on :27017

### Build & Deployment
- `npm run build` - Creates optimized production build in `build/` folder
- Frontend is CRA (Create React App) based

## Project-Specific Conventions & Gotchas

### Hardcoded JWT Token
- **Location**: `src/context/notes/NoteState.js` lines 12, 34, 51, 68
- **Issue**: Same JWT token hardcoded in all API calls - NOT production-ready
- **TODO**: Extract token from localStorage after login, pass dynamically in headers
- **Format**: `Bearer <token>` pattern expected by `fetchuser` middleware

### Context Immutability
- **Pattern in `NoteState.js`**: Use `JSON.parse(JSON.stringify(notes))` before mutation (line 81) to prevent reference issues
- **Reason**: React state updates must be immutable for proper re-renders

### Modal Management
- **Notes.js**: Uses hidden button + Bootstrap modal (`data-bs-toggle`, `data-bs-target`) instead of controlled modal component
- **Refs**: `useRef()` used to trigger modal and handle form submission

### API Request Headers
All API calls in `NoteState.js` include:
```javascript
headers: {
  'Content-Type': 'application/json',
  'auth-token': '<jwt-token>'
}
```

### Input Validation
- **Frontend**: HTML5 attributes (`minLength={5}`) on form inputs
- **Backend**: `express-validator` rules in route handlers (see `auth.js` line 11-14, `notes.js` line 17-18)

## Component Communication

### Example: Edit Note Flow
1. User clicks edit → `updateNote()` in `Notes.js` (line 21) sets modal state
2. User submits → `handleClick()` calls `editNote()` from context
3. `editNote()` in `NoteState.js` (line 64):
   - Makes PUT request to `/api/notes/updatenote/:id`
   - Updates local state by finding and replacing note object
   - Backend validates, updates MongoDB, returns updated note

## Files to Reference When Making Changes

- **Adding new note fields**: Update `Note.js` model + `NoteState.js` CRUD methods + backend route validation
- **Changing authentication**: Modify `backend/routes/auth.js`, `backend/middleware/fetchuser.js`, and token handling in `NoteState.js`
- **Adding routes**: Extend `App.js` Switch/Route, create component in `src/components/`, add backend route file
- **Styling**: Global CSS in `src/App.css` and `src/index.css` (Bootstrap via CDN in `public/index.html`)

## Known Issues to Avoid

- JWT token is hardcoded (security issue)
- No error handling in API calls - responses not checked for errors
- Modal state uses refs instead of component state
- No loading/error states for async operations
