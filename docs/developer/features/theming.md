# Téma és Lokalizáció

A FormFiller támogatja a világos/sötét témákat és többnyelvű felhasználói felületet.

## Téma Kezelés

### Elérhető Témák

A rendszer DevExtreme témákat használ:

| Téma | Leírás |
|------|--------|
| `generic.light` | Világos téma |
| `generic.dark` | Sötét téma |
| `material.blue.light` | Material Design világos |
| `material.blue.dark` | Material Design sötét |

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
    // Mentett preferencia vagy rendszer beállítás
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

### Téma Alkalmazás

```typescript
// utils/theme.ts
export function applyTheme(theme: 'light' | 'dark') {
  const root = document.documentElement;
  
  // CSS osztály beállítás
  root.classList.remove('theme-light', 'theme-dark');
  root.classList.add(`theme-${theme}`);
  
  // DevExtreme téma váltás
  const dxTheme = theme === 'dark' ? 'generic.dark' : 'generic.light';
  
  // Dinamikus CSS betöltés
  const existingLink = document.getElementById('dx-theme');
  if (existingLink) {
    existingLink.remove();
  }
  
  const link = document.createElement('link');
  link.id = 'dx-theme';
  link.rel = 'stylesheet';
  link.href = `/themes/dx.${dxTheme}.css`;
  document.head.appendChild(link);
  
  // Meta theme-color (mobil böngészőkhöz)
  document.querySelector('meta[name="theme-color"]')?.setAttribute(
    'content',
    theme === 'dark' ? '#1a1a1a' : '#ffffff'
  );
}
```

### Téma Váltó Komponens

```typescript
import { useTheme } from '../contexts/ThemeContext';

function ThemeSwitcher() {
  const { theme, toggleTheme } = useTheme();

  return (
    <Button
      icon={theme === 'dark' ? 'sun' : 'moon'}
      onClick={toggleTheme}
      hint={theme === 'dark' ? 'Világos téma' : 'Sötét téma'}
    />
  );
}
```

### CSS Változók

```scss
// styles/variables.scss
:root {
  // Világos téma (alapértelmezett)
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

// Használat
.my-component {
  background: var(--bg-primary);
  color: var(--text-primary);
  border: 1px solid var(--border-color);
}
```

## Lokalizáció (i18n)

### i18next Konfiguráció

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
    fallbackLng: 'hu',
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

### Fordítási Fájlok

```json
// locales/hu.json
{
  "common": {
    "save": "Mentés",
    "cancel": "Mégse",
    "delete": "Törlés",
    "edit": "Szerkesztés",
    "loading": "Betöltés...",
    "error": "Hiba történt"
  },
  "auth": {
    "login": "Bejelentkezés",
    "logout": "Kijelentkezés",
    "register": "Regisztráció",
    "forgotPassword": "Elfelejtett jelszó"
  },
  "form": {
    "required": "Kötelező mező",
    "invalidEmail": "Érvénytelen email cím",
    "saved": "Sikeresen mentve"
  },
  "validation": {
    "required": "{{field}} megadása kötelező",
    "minLength": "Minimum {{min}} karakter szükséges",
    "maxLength": "Maximum {{max}} karakter engedélyezett"
  }
}
```

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
  }
}
```

### Fordítások Használata

```typescript
import { useTranslation } from 'react-i18next';

function MyComponent() {
  const { t } = useTranslation();

  return (
    <div>
      <h1>{t('common.save')}</h1>
      
      {/* Interpoláció */}
      <p>{t('validation.required', { field: 'Email' })}</p>
      
      {/* Többes szám */}
      <p>{t('items', { count: 5 })}</p>
    </div>
  );
}
```

### Nyelv Váltó

```typescript
function LanguageSwitcher() {
  const { i18n } = useTranslation();
  
  const languages = [
    { code: 'hu', name: 'Magyar', flag: '🇭🇺' },
    { code: 'en', name: 'English', flag: '🇬🇧' }
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

### DevExtreme Lokalizáció

```typescript
// DevExtreme komponensek lokalizációja
import { locale, loadMessages } from 'devextreme/localization';
import huMessages from 'devextreme/localization/messages/hu.json';

// Üzenetek betöltése
loadMessages(huMessages);

// Nyelv beállítása
locale('hu');
```

### Dátum és Szám Formázás

```typescript
import { formatDate, formatNumber } from 'devextreme/localization';

// Dátum formázás
const formattedDate = formatDate(new Date(), 'shortDate'); // 2024.01.15.

// Szám formázás
const formattedNumber = formatNumber(1234.56, { type: 'currency', currency: 'HUF' }); // 1 234,56 Ft
```

## Felhasználói Preferenciák

### Preferencia Mentés

```typescript
interface UserPreferences {
  theme: 'light' | 'dark';
  language: string;
  dateFormat: string;
  notifications: boolean;
}

function PreferencesPage() {
  const { user, updatePreferences } = useAuth();
  const { theme, setTheme } = useTheme();
  const { i18n } = useTranslation();

  const [preferences, setPreferences] = useState<UserPreferences>(
    user?.preferences || defaultPreferences
  );

  const handleSave = async () => {
    await updatePreferences(preferences);
    
    // Lokális alkalmazás
    setTheme(preferences.theme);
    i18n.changeLanguage(preferences.language);
    
    notify(t('common.saved'), 'success');
  };

  return (
    <Form formData={preferences}>
      <GroupItem caption={t('preferences.appearance')}>
        <SimpleItem
          dataField="theme"
          editorType="dxSelectBox"
          editorOptions={{
            items: [
              { value: 'light', text: t('preferences.lightTheme') },
              { value: 'dark', text: t('preferences.darkTheme') }
            ],
            valueExpr: 'value',
            displayExpr: 'text'
          }}
        />
        <SimpleItem
          dataField="language"
          editorType="dxSelectBox"
          editorOptions={{
            items: [
              { value: 'hu', text: 'Magyar' },
              { value: 'en', text: 'English' }
            ],
            valueExpr: 'value',
            displayExpr: 'text'
          }}
        />
      </GroupItem>
      
      <ButtonItem>
        <ButtonOptions text={t('common.save')} onClick={handleSave} />
      </ButtonItem>
    </Form>
  );
}
```

## Rendszer Preferencia Követés

```typescript
// Rendszer sötét mód változás figyelése
useEffect(() => {
  const mediaQuery = window.matchMedia('(prefers-color-scheme: dark)');
  
  const handleChange = (e: MediaQueryListEvent) => {
    // Csak ha nincs explicit felhasználói beállítás
    if (!localStorage.getItem('theme')) {
      setTheme(e.matches ? 'dark' : 'light');
    }
  };

  mediaQuery.addEventListener('change', handleChange);
  return () => mediaQuery.removeEventListener('change', handleChange);
}, []);
```

## Animációk és Téma

```typescript
// Animáció preferencia
const prefersReducedMotion = window.matchMedia('(prefers-reduced-motion: reduce)').matches;

// Feltételes animáció
const animationConfig = prefersReducedMotion
  ? { duration: 0 }
  : { duration: 300, easing: 'ease-out' };
```

## Site-Specifikus Branding

```typescript
function useSiteBranding() {
  const { currentSite } = useSite();
  const { theme } = useTheme();

  useEffect(() => {
    if (currentSite?.branding) {
      const { branding } = currentSite;
      const root = document.documentElement;

      // Site-specifikus színek
      if (branding.primaryColor) {
        root.style.setProperty('--site-primary', branding.primaryColor);
      }
      
      if (branding.secondaryColor) {
        root.style.setProperty('--site-secondary', branding.secondaryColor);
      }

      // Logo
      if (branding.logo) {
        root.style.setProperty('--site-logo', `url(${branding.logo})`);
      }
    }
  }, [currentSite, theme]);
}
```

## Best Practices

### 1. Kerüld a Hardcoded Szövegeket

```typescript
// ❌ Rossz
<Button text="Mentés" />

// ✅ Jó
<Button text={t('common.save')} />
```

### 2. Használj CSS Változókat

```scss
// ❌ Rossz
.button {
  background: #1976d2;
}

// ✅ Jó
.button {
  background: var(--accent-color);
}
```

### 3. Teszteld Mindkét Témában

A fejlesztés során rendszeresen válts témát, hogy minden jól nézzen ki mindkét módban.

### 4. RTL Támogatás (Jövőbeli)

Ha szükséges jobbról-balra írás támogatása:

```typescript
const direction = isRTL(language) ? 'rtl' : 'ltr';
document.documentElement.setAttribute('dir', direction);
```

