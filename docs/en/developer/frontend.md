# Frontend Development

## Overview

The FormFiller frontend is a modern, React and TypeScript based single-page application (SPA) that dynamically renders forms, data grids, and tree structures based on JSON configuration.

### Main Advantages

| Advantage | Description |
|-----------|-------------|
| **Configuration-driven** | No hardcoded UI - everything comes from the schema |
| **Multiple render engines** | DevExtreme, Material-UI, Print - same config |
| **Reactive** | Fields automatically react to each other |
| **Type-safe** | Full TypeScript coverage |
| **Modular** | Easy to extend with new components |
| **Offline-capable** | Optimistic UI, background synchronization |

### Schematic Structure

```mermaid
flowchart TB
    subgraph App["FRONTEND APPLICATION"]
        subgraph Pages["PAGES"]
            P1[Home]
            P2[Form]
            P3[Results]
            P4[Profile]
            P5[Admin]
            P6[Tasks]
        end
        
        subgraph Components["COMPONENTS"]
            C1[FormComponent]
            C2[GridView]
            C3[TreeView]
            
            subgraph Renderers["RENDERER FACTORIES"]
                R1[DevExtreme]
                R2[Material-UI]
                R3[Print]
            end
        end
        
        subgraph Managers["MANAGERS & SERVICES"]
            M1[FormManager]
            M2[EventHandler Registry]
            M3[DataService]
        end
        
        subgraph Contexts["CONTEXTS"]
            CTX1[AuthContext]
            CTX2[NavigationContext]
            CTX3[ThemeContext]
        end
    end
    
    API[BACKEND API]
    
    Pages --> Components
    C1 --> Renderers
    C2 --> Renderers
    C3 --> Renderers
    Components --> Managers
    Managers --> Contexts
    Contexts --> API
```

### Call Chains

#### Form Display

```mermaid
flowchart TB
    URL["URL: /form/:configId/:recordId"]
    FP["FormPage<br/>← React Router"]
    CS["configService"]
    BE["Backend<br/>GET /api/config/:configId"]
    FC["FormComponent<br/>← Config processing"]
    FM["FormManager<br/>← Field registration"]
    RF["Renderer Factory<br/>← DevExtreme/MUI/Print"]
    DX["DevExtreme Components<br/>← UI rendering"]
    
    URL --> FP
    FP --> CS
    CS --> BE
    CS --> FC
    FC --> FM
    FM --> RF
    RF --> DX
```

#### Field Value Change

```mermaid
flowchart TB
    U["User enters value"]
    DX["DevExtreme Editor<br/>onValueChanged"]
    FM["FormManager<br/>handleFieldChange"]
    EHR["EventHandlerRegistry<br/>(onValueChanged)"]
    FUS["FieldUpdateService<br/>(visibleIf, etc.)"]
    H["Handlers run:<br/>calculate, setValue, validate"]
    F["Other fields update<br/>automatically"]
    
    U --> DX
    DX --> FM
    FM --> EHR
    FM --> FUS
    EHR --> H
    FUS --> F
```

#### Save Process

```mermaid
flowchart TB
    U["User clicks 'Save'"]
    FM["FormManager<br/>collectData()"]
    V["Validator<br/>validate(data)"]
    VAL{Validation}
    ERR["Error message<br/>display"]
    DS["dataService<br/>save(configId, data)"]
    API["Backend API<br/>POST /api/data/..."]
    OK["Successful save<br/>Toast notification"]
    
    U --> FM
    FM --> V
    V --> VAL
    VAL -->|INVALID| ERR
    VAL -->|VALID| DS
    DS --> API
    API --> OK
```

## Detailed Documentation

Detailed descriptions of specific features in separate documents:

- [User Management](./features/user-management.md) - Registration, login, profile
- [Access Control (RBAC)](./features/rbac.md) - Roles, permissions, UI filtering
- [Multisite Management](./features/multisite.md) - Multi-tenant operation on frontend
- [Theme and Localization](./features/theming.md) - Themes, multi-language

## Architecture

The frontend is React and TypeScript based, with Vite build system and DevExtreme UI components.

## Project Structure

```
src/
├── components/          # React components
│   ├── form/           # Form components
│   ├── header/         # Header
│   ├── footer/         # Footer
│   └── views/          # View modules (Grid, Tree)
├── pages/              # Page components
│   ├── home/           # Home page
│   ├── form/           # Form page
│   ├── results/        # Results page
│   └── profile/        # Profile page
├── services/           # Business logic and API
│   ├── dataService.ts  # API calls
│   └── EventHandlerRegistry.ts
├── factories/          # Renderer factories
├── managers/           # State managers
├── eventHandlers/      # Event handlers
├── contexts/           # React contexts
├── interfaces/         # TypeScript interfaces
├── types/              # Type definitions
├── utils/              # Utility functions
└── themes/             # DevExtreme themes
```

## Renderers

The system supports three rendering engines:

### DevExtreme Renderer

The main rendering engine with full functionality:

```typescript
import { DevExtremeFormRenderer } from './factories/DevExtremeFormRenderer';

const renderer = new DevExtremeFormRenderer();
const form = renderer.render(config, data, mode);
```

### Material-UI Renderer

Alternative Material Design appearance:

```typescript
import { MUIFormRenderer } from './factories/MUIFormRenderer';
```

### Print Renderer

Print-optimized view.

## Form Manager

Central state manager for forms:

```typescript
import { FormManager } from './managers/FormManager';

const formManager = new FormManager();

// Register field
formManager.registerField('firstName', {
  value: '',
  onChange: (value) => console.log('Changed:', value)
});

// Set value
formManager.setValue('firstName', 'John');

// Collect data
const formData = formManager.collectData();
```

## Event Handling

Declarative event handling system:

```typescript
// Configuration
const fieldConfig = {
  name: 'quantity',
  type: 'number',
  onValueChanged: [
    { handler: 'log', params: { message: 'Quantity changed' } },
    { handler: 'calculate', params: { target: 'total', formula: 'quantity * price' } }
  ]
};

// Register custom handler
EventHandlerRegistry.register('myHandler', (context, params) => {
  // Handler implementation
});
```

## API Calls

The `dataService` handles API communication:

```typescript
import { configService, dataService } from './services/dataService';

// Get configuration
const config = await configService.getById(configId);

// Get data
const data = await dataService.query(configId, { filter, sort, skip, take });

// Save
await dataService.save(configId, formData);
```

## Contexts

### Auth Context

```typescript
import { useAuth } from './contexts/AuthContext';

const { user, login, logout, isAuthenticated } = useAuth();
```

### Navigation Context

```typescript
import { useNavigation } from './contexts/NavigationContext';

const { navigate, currentRoute } = useNavigation();
```

## Component Development

### Creating New Component

```typescript
// components/MyComponent/MyComponent.tsx
import React from 'react';
import './MyComponent.scss';

interface MyComponentProps {
  title: string;
  onAction: () => void;
}

export const MyComponent: React.FC<MyComponentProps> = ({ title, onAction }) => {
  return (
    <div className="my-component">
      <h2>{title}</h2>
      <button onClick={onAction}>Action</button>
    </div>
  );
};
```

## Themes

DevExtreme themes in the `themes/` directory:

```bash
# Theme build
npm run build-themes
```

## Useful Commands

```bash
# Development server
npm start

# Build
npm run build

# Tests
npm test

# Theme build
npm run build-themes
```

