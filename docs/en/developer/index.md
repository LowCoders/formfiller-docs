# Developer Documentation

This documentation is intended for developers of the FormFiller system.

## Contents

### General

- [Backend Development](./backend.md) - Backend architecture and development guide
- [Frontend Development](./frontend.md) - Frontend architecture, call chains, advantages
- [Schema and Types](./schema.md) - Low-code definition language, shared types

### API and Integration

- [API Reference](./api-reference.md) - REST API documentation
- [Event Handling](./event-handling.md) - Declarative event handling system

### Components

- [Form Components](./form-components.md) - Form components and renderers
- [Validation](./validation.md) - Validation system, group validators, computedRules

### Features (detailed)

- [User Management](./features/user-management.md) - Registration, login, profile, token handling
- [Access Control (RBAC)](./features/rbac.md) - Roles, permissions, UI filtering
- [Multisite Management](./features/multisite.md) - Multi-tenant operation, site context
- [Theme and Localization](./features/theming.md) - Themes, multi-language, i18n
- [Workflow Management](./features/workflow.md) - Business processes, step types, error handling
- [Data Management](./features/data-management.md) - Save, query, export, save limit
- [🤖 AI Interface](./features/ai-interface.md) - **Working feature!** Natural language generation, ~98% time savings

## Development Environment Setup

### Prerequisites

- Node.js 18+
- MongoDB 4.4+ (locally or in Docker)
- Git

### Cloning Repositories

```bash
# Create main directory
mkdir formfiller && cd formfiller

# Clone repositories
git clone <repo-url>/formfiller-backend
git clone <repo-url>/formfiller-frontend
git clone <repo-url>/formfiller-schema
git clone <repo-url>/formfiller-validator
git clone <repo-url>/formfiller-types
git clone <repo-url>/formfiller-deployment
```

### Schema Setup (first)

```bash
cd formfiller-schema
npm install
npm run build
npm run distribute  # Distribute to other projects
```

### Starting Backend

```bash
cd formfiller-backend
npm install
cp env.example .env
# Edit the .env file
npm run dev
```

### Starting Frontend

```bash
cd formfiller-frontend
npm install
cp .env.development.example .env.development
npm start
```

## Development Practices

### Code Style

- TypeScript strict mode
- Use ESLint and Prettier
- English comments and variable names

### Git Workflow

1. Create feature branch: `feature/feature-name`
2. Commit message format: `type: description`
   - `feat:` - New feature
   - `fix:` - Bug fix
   - `docs:` - Documentation
   - `refactor:` - Code restructuring
   - `test:` - Tests
3. Pull request to `develop` branch

### Testing

```bash
# Backend tests
cd formfiller-backend
npm test

# Frontend tests
cd formfiller-frontend
npm test

# Validator tests
cd formfiller-validator
npm test
```

## Troubleshooting

### MongoDB Connection

If MongoDB is unreachable:
```bash
# With Docker
docker run -d -p 27017:27017 --name mongodb mongo:7

# Or with brew (macOS)
brew services start mongodb-community
```

### Schema Changes

If the schema has been modified, it needs to be redistributed:
```bash
cd formfiller-schema
npm run distribute
# Then restart backend and frontend projects
```

### Port Conflicts

- Backend: 3001 (configurable in .env)
- Frontend: 3000 (configurable in vite.config.ts)
- MongoDB: 27017
- Redis: 6379 (optional)

