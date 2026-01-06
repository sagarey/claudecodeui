# AGENTS.md - Claude Code UI Development Guide

## Build, Lint, and Test Commands

### Development
```bash
npm run dev          # Start both backend (3001) and frontend (5173) with hot reload
npm run server       # Start backend only
npm run client       # Start frontend Vite dev server only
```

### Build and Deployment
```bash
npm run build        # Production build to dist/ (Vite builds React app)
npm run preview      # Preview production build locally
npm run start        # Build then start production server
npm run release      # Run release.sh script for npm publishing
```

### No Test or Lint Commands
This project has no configured test framework or linting. Consider adding:
- Vitest for unit tests
- ESLint + Prettier for code quality

## Code Style Guidelines

### General Principles
- Use ES modules (`import`/`export`) throughout
- No TypeScript - plain JavaScript only
- Avoid unnecessary comments; document complex logic only
- Keep files focused - split large components

### Import Ordering
1. React core imports (`react`)
2. Third-party libraries (alphabetical)
3. Local components
4. Local utils/helpers
5. Context hooks
```jsx
import React, { useState, useEffect, useRef } from 'react';
import { useDropzone } from 'react-dropzone';
import { FolderOpen, Plus } from 'lucide-react';
import Sidebar from './Sidebar';
import { api, authenticatedFetch } from '../utils/api';
import { useAuth } from '../contexts/AuthContext';
```

### React Components
- Use functional components with hooks
- Default export for page/feature components
- Named exports for reusable components
- Use `memo()` for expensive components
- Prefer `useCallback` and `useMemo` for performance
```jsx
import React, { memo } from 'react';

const Button = ({ children, onClick }) => {
  return <button onClick={onClick}>{children}</button>;
};

export default memo(Button);
```

### Context Pattern
```jsx
import React, { createContext, useContext } from 'react';

const MyContext = createContext(null);

export const useMyContext = () => {
  const context = useContext(MyContext);
  if (!context) {
    throw new Error('useMyContext must be used within MyProvider');
  }
  return context;
};

export const MyProvider = ({ children }) => {
  return <MyContext.Provider value={value}>{children}</MyContext.Provider>;
};
```

### Naming Conventions
| Pattern | Example | Rule |
|---------|---------|------|
| Components | `ChatInterface.jsx` | PascalCase |
| Hooks | `useWebSocket()` | camelCase, "use" prefix |
| Utils/Functions | `formatTimeAgo()` | camelCase |
| Contexts | `AuthContext.jsx` | PascalCase, "Context" suffix |
| Constants | `MAX_FILE_SIZE` | UPPER_SNAKE_CASE |
| Props | `onProjectSelect` | camelCase, event handlers with "on" prefix |

### Error Handling
- Wrap async operations in try-catch
- Use meaningful error messages
- Log errors to console with context
```javascript
try {
  const result = await api.projects();
  return result;
} catch (error) {
  console.error('Failed to fetch projects:', error);
  throw new Error('Unable to load projects');
}
```

### Tailwind CSS
- Use utility classes from Tailwind
- Follow shadcn/ui patterns for components
- Use `cn()` utility for class merging
```jsx
import { cn } from '../lib/utils';

<div className={cn(
  "flex items-center gap-2",
  isActive && "bg-primary text-primary-foreground"
)}>
```

### Environment Variables
- Frontend: `import.meta.env.VITE_*`
- Backend: `process.env.*`
- Never expose secrets to frontend

### File Structure
```
src/
├── components/     # React components
├── contexts/       # React context providers
├── utils/          # Utility functions
├── lib/            # Configuration (utils.js, utils.ts)
├── App.jsx         # Root component
└── main.jsx        # Entry point

server/
├── index.js        # Express + WebSocket entry
├── routes/         # API route handlers
├── middleware/     # Express middleware
└── database/       # SQLite database
```

### WebSocket Communication
- Chat messages: `/ws` endpoint
- Terminal: `/shell` endpoint (node-pty)
- Message types: `claude-command`, `cursor-command`, `session-created`, `claude-complete`

### Session Protection Pattern
When user sends message, mark session active:
```javascript
activeSessions.add(sessionId);
```
On conversation complete/abort:
```javascript
activeSessions.delete(sessionId);
```
This prevents sidebar updates from interrupting active conversations.
