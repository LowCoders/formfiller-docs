# Fejlesztői Dokumentáció

Ez a dokumentáció a FormFiller rendszer fejlesztőinek szól.

## Tartalom

### Általános

- [Backend Fejlesztés](./backend.md) - Backend architektúra és fejlesztési útmutató
- [Frontend Fejlesztés](./frontend.md) - Frontend architektúra, hívási láncok, előnyök
- [Schema és Típusok](./schema.md) - Low-code definíciós nyelv, közös típusok

### API és Integráció

- [API Referencia](./api-reference.md) - REST API dokumentáció
- [Eseménykezelés](./event-handling.md) - Deklaratív eseménykezelő rendszer

### Komponensek

- [Form Komponensek](./form-components.md) - Űrlap komponensek és rendererek
- [Validáció](./validation.md) - Validációs rendszer, group validátorok, computedRules

### Funkciók (részletes)

- [Felhasználó Kezelés](./features/user-management.md) - Regisztráció, bejelentkezés, profil, token kezelés
- [Jogosultságkezelés (RBAC)](./features/rbac.md) - Szerepkörök, engedélyek, UI szűrés
- [Multisite Kezelés](./features/multisite.md) - Több bérlős működés, site kontextus
- [Téma és Lokalizáció](./features/theming.md) - Témák, többnyelvűség, i18n
- [Workflow Kezelés](./features/workflow.md) - Üzleti folyamatok, step típusok, hibakezelés
- [Adatkezelés](./features/data-management.md) - Mentés, lekérdezés, exportálás, save limit
- [🤖 AI Interfész](./features/ai-interface.md) - **Működő funkció!** Természetes nyelvű generálás, ~98% időmegtakarítás

## Fejlesztői Környezet Beállítása

### Előfeltételek

- Node.js 18+
- MongoDB 4.4+ (lokálisan vagy Docker-ben)
- Git

### Repók Klónozása

```bash
# Fő könyvtár létrehozása
mkdir formfiller && cd formfiller

# Repók klónozása
git clone <repo-url>/formfiller-backend
git clone <repo-url>/formfiller-frontend
git clone <repo-url>/formfiller-schema
git clone <repo-url>/formfiller-validator
git clone <repo-url>/formfiller-types
git clone <repo-url>/formfiller-deployment
```

### Schema Beállítása (első)

```bash
cd formfiller-schema
npm install
npm run build
npm run distribute  # Disztribúció a többi projektbe
```

### Backend Indítása

```bash
cd formfiller-backend
npm install
cp env.example .env
# Szerkeszd a .env fájlt
npm run dev
```

### Frontend Indítása

```bash
cd formfiller-frontend
npm install
cp .env.development.example .env.development
npm start
```

## Fejlesztési Gyakorlatok

### Kód Stílus

- TypeScript strict mode
- ESLint és Prettier használata
- Angol nyelvű kommentek és változónevek

### Git Workflow

1. Feature branch létrehozása: `feature/feature-name`
2. Commit message formátum: `type: description`
   - `feat:` - Új funkció
   - `fix:` - Hibajavítás
   - `docs:` - Dokumentáció
   - `refactor:` - Kód átszervezés
   - `test:` - Tesztek
3. Pull request a `develop` branch-be

### Tesztelés

```bash
# Backend tesztek
cd formfiller-backend
npm test

# Frontend tesztek
cd formfiller-frontend
npm test

# Validator tesztek
cd formfiller-validator
npm test
```

## Hibaelhárítás

### MongoDB Kapcsolat

Ha a MongoDB nem elérhető:
```bash
# Docker-rel
docker run -d -p 27017:27017 --name mongodb mongo:7

# Vagy brew-vel (macOS)
brew services start mongodb-community
```

### Schema Változások

Ha a schema módosult, újra kell disztributálni:
```bash
cd formfiller-schema
npm run distribute
# Majd újraindítás a backend és frontend projektekben
```

### Port Ütközések

- Backend: 3001 (módosítható .env-ben)
- Frontend: 3000 (módosítható vite.config.ts-ben)
- MongoDB: 27017
- Redis: 6379 (opcionális)

