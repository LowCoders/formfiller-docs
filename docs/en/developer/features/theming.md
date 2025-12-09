# Theme and Localization

FormFiller supports light/dark themes and multi-language user interface.

## Theme Management

### Available Themes

The system uses DevExtreme themes:

| Theme | Description |
|-------|-------------|
| `generic.light` | Light theme |
| `generic.dark` | Dark theme |
| `material.blue.light` | Material Design light |
| `material.blue.dark` | Material Design dark |

### ThemeContext

```typescript
interface ThemeContextType {
  theme: 'light' | 'dark';
  setTheme: (theme: 'light' | 'dark') => void;
  toggleTheme: () => void;
}

const ThemeContext = createContext<ThemeContextType | null>(null);

export function ThemeProvider({ children }: { children: React.ReactNode }) {
  const [theme, setThemeState] = useState<'light' | 'dark'>(() => {
    // Saved preference or system setting
    const saved = localStorage.getItem('theme');
    if (saved) return saved as 'light' | 'dark';
    
    return window.matchMedia('(prefers-color-scheme: dark)').matches 
      ? 'dark' 
      : 'light';
  });

  const setTheme = (newTheme: 'light' | 'dark') => {
    setThemeState(newTheme);
    localStorage.setItem('theme', newTheme);
    applyTheme(newTheme);
  };

  const toggleTheme = () => {
    setTheme(theme === 'light' ? 'dark' : 'light');
  };

  useEffect(() => {
    applyTheme(theme);
  }, []);

  return (
    <ThemeContext.Provider value={{ theme, setTheme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}
```

### Theme Switcher Component

```typescript
import { useTheme } from '../contexts/ThemeContext';

function ThemeSwitcher() {
  const { theme, toggleTheme } = useTheme();

  return (
    <Button
      icon={theme === 'dark' ? 'sun' : 'moon'}
      onClick={toggleTheme}
      hint={theme === 'dark' ? 'Light theme' : 'Dark theme'}
    />
  );
}
```

### CSS Variables

```scss
// styles/variables.scss
:root {
  // Light theme (default)
  --bg-primary: #ffffff;
  --bg-secondary: #f5f5f5;
  --text-primary: #333333;
  --text-secondary: #666666;
  --border-color: #e0e0e0;
  --accent-color: #1976d2;
}

.theme-dark {
  --bg-primary: #1a1a1a;
  --bg-secondary: #2d2d2d;
  --text-primary: #ffffff;
  --text-secondary: #b0b0b0;
  --border-color: #404040;
  --accent-color: #90caf9;
}

// Usage
.my-component {
  background: var(--bg-primary);
  color: var(--text-primary);
  border: 1px solid var(--border-color);
}
```

## Localization (i18n)

### i18next Configuration

```typescript
// i18n/config.ts
import i18n from 'i18next';
import { initReactI18next } from 'react-i18next';
import LanguageDetector from 'i18next-browser-languagedetector';

import hu from './locales/hu.json';
import en from './locales/en.json';

i18n
  .use(LanguageDetector)
  .use(initReactI18next)
  .init({
    resources: {
      hu: { translation: hu },
      en: { translation: en }
    },
    fallbackLng: 'en',
    interpolation: {
      escapeValue: false
    },
    detection: {
      order: ['localStorage', 'navigator'],
      caches: ['localStorage']
    }
  });

export default i18n;
```

### Translation Files

```json
// locales/en.json
{
  "common": {
    "save": "Save",
    "cancel": "Cancel",
    "delete": "Delete",
    "edit": "Edit",
    "loading": "Loading...",
    "error": "An error occurred"
  },
  "auth": {
    "login": "Login",
    "logout": "Logout",
    "register": "Register",
    "forgotPassword": "Forgot Password"
  },
  "form": {
    "required": "Required field",
    "invalidEmail": "Invalid email address",
    "saved": "Successfully saved"
  },
  "validation": {
    "required": "{{field}} is required",
    "minLength": "Minimum {{min}} characters required",
    "maxLength": "Maximum {{max}} characters allowed"
  }
}
```

### Using Translations

```typescript
import { useTranslation } from 'react-i18next';

function MyComponent() {
  const { t } = useTranslation();

  return (
    <div>
      <h1>{t('common.save')}</h1>
      
      {/* Interpolation */}
      <p>{t('validation.required', { field: 'Email' })}</p>
      
      {/* Pluralization */}
      <p>{t('items', { count: 5 })}</p>
    </div>
  );
}
```

### Language Switcher

```typescript
function LanguageSwitcher() {
  const { i18n } = useTranslation();
  
  const languages = [
    { code: 'en', name: 'English', flag: '🇬🇧' },
    { code: 'hu', name: 'Magyar', flag: '🇭🇺' }
  ];

  return (
    <SelectBox
      items={languages}
      valueExpr="code"
      displayExpr="name"
      value={i18n.language}
      onValueChanged={e => i18n.changeLanguage(e.value)}
      itemRender={item => (
        <span>{item.flag} {item.name}</span>
      )}
    />
  );
}
```

### DevExtreme Localization

```typescript
// DevExtreme component localization
import { locale, loadMessages } from 'devextreme/localization';
import enMessages from 'devextreme/localization/messages/en.json';

// Load messages
loadMessages(enMessages);

// Set language
locale('en');
```

## Best Practices

### 1. Avoid Hardcoded Text

```typescript
// ❌ Bad
<Button text="Save" />

// ✅ Good
<Button text={t('common.save')} />
```

### 2. Use CSS Variables

```scss
// ❌ Bad
.button {
  background: #1976d2;
}

// ✅ Good
.button {
  background: var(--accent-color);
}
```

### 3. Test in Both Themes

During development, regularly switch themes to ensure everything looks good in both modes.

