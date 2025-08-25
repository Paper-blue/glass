# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Glass is an Electron-based desktop application with a Next.js web dashboard that acts as a digital mind extension. It captures screen content and audio in real-time, provides AI-powered summaries and insights, and maintains both local (SQLite) and cloud (Firebase) data storage.

## Common Development Commands

### Initial Setup and Development
```bash
# Full setup (installs dependencies for both main app and web dashboard)
npm run setup

# Start development (builds renderer and starts Electron)
npm start

# Development with live reload
npm run watch:renderer
```

### Build Commands
```bash
# Build all components (renderer + web dashboard)
npm run build:all

# Build for Windows
npm run build:win

# Build for distribution
npm run build

# Build only the renderer
npm run build:renderer

# Build only the web dashboard
npm run build:web
```

### Quality Checks
```bash
# Run linting
npm run lint

# Web dashboard specific commands (run from pickleglass_web/)
cd pickleglass_web
npm run dev    # Start Next.js dev server
npm run build  # Build Next.js app
npm run lint   # Lint web dashboard
```

## Architecture Overview

### Core Design Principles

1. **Service-Repository Pattern**: All business logic resides in Services, data access in Repositories
2. **Dual Database Architecture**: Automatic switching between SQLite (offline) and Firebase (online)
3. **Centralized Data Logic**: All database operations MUST go through the Electron main process
4. **Feature-Based Organization**: Code organized by features in `src/features/`
5. **AI Provider Abstraction**: Factory pattern for AI providers in `src/features/common/ai/`

### Project Structure

```
glass/
├── src/                          # Electron main application
│   ├── features/                 # Feature modules
│   │   ├── ask/                 # Q&A feature
│   │   ├── listen/              # Audio capture & transcription
│   │   │   ├── stt/            # Speech-to-text
│   │   │   └── summary/        # AI summaries
│   │   ├── settings/            # User settings
│   │   ├── shortcuts/           # Keyboard shortcuts
│   │   └── common/              # Shared modules
│   │       ├── ai/              # AI provider factory
│   │       │   └── providers/  # Individual AI providers
│   │       ├── config/          # App configuration
│   │       ├── repositories/    # Data access layer
│   │       └── services/        # Business logic
│   ├── bridge/                  # IPC communication bridges
│   ├── ui/                      # Electron renderer UI
│   └── window/                  # Window management
├── pickleglass_web/             # Next.js web dashboard
│   ├── app/                     # Next.js app router pages
│   ├── backend_node/            # Express.js backend
│   │   └── ipcBridge.js        # IPC communication with Electron
│   └── utils/                   # Frontend utilities
└── aec/                         # Audio echo cancellation (Rust)
```

### Repository Pattern Implementation

Every repository dealing with user data has dual implementations:

1. **SQLite Repository** (`*.sqlite.repository.js`): Local database operations
2. **Firebase Repository** (`*.firebase.repository.js`): Cloud database operations
3. **Factory/Adapter** (`index.js`): Automatically selects repository based on auth state

The factory pattern in each repository's `index.js`:
- Checks authentication status via `authService.getCurrentUser()`
- Selects appropriate repository (SQLite for offline, Firebase for authenticated)
- Injects user ID automatically for Firebase operations

### IPC Communication Flow

Web Dashboard → Electron Main Process:
1. Next.js frontend makes API call to Express backend
2. Express backend uses `ipcBridge.js` to send IPC request
3. Electron main process receives request and executes via Service/Repository
4. Data returned through IPC response channel
5. Express backend returns HTTP response to frontend

### AI Provider Integration

To add a new AI provider:
1. Create provider module in `src/features/common/ai/providers/`
2. Implement the standard interface (see existing providers)
3. Register in `src/features/common/ai/factory.js`

### Security Considerations

- All sensitive data is encrypted before Firebase storage using `createEncryptedConverter`
- API keys stored securely using Electron's native keytar integration
- No direct database access from web dashboard (must go through IPC)

## Development Guidelines

1. **Never access databases directly from UI layers** - Always use Services
2. **Follow the existing patterns** - Check similar features before implementing
3. **Maintain dual repository implementations** - Both SQLite and Firebase versions required
4. **Use feature-based organization** - Keep related code together in feature folders
5. **Test both online and offline modes** - Ensure seamless database switching
6. **Run linting before commits** - Use `npm run lint` to check code style

## Key Files to Understand

- `src/features/common/services/sqliteClient.js`: SQLite database client and schema
- `src/features/common/services/firebaseClient.js`: Firebase client and configuration
- `src/features/common/services/authService.js`: Authentication and user management
- `pickleglass_web/backend_node/ipcBridge.js`: IPC communication bridge
- `src/features/common/ai/factory.js`: AI provider factory
- `docs/DESIGN_PATTERNS.md`: Detailed architectural documentation