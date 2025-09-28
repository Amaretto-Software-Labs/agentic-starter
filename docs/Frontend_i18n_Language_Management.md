# Frontend i18n & Language Management Guide

**Date:** 2025-09-24
**Status:** Active Implementation
**Scope:** React Web Application (apps/web)

This document explains how to manage, configure, and extend the internationalization (i18n) system.

---

## Overview

The frontend uses **React i18next** with custom language detection logic for client-side internationalization. This system handles:
- Static UI text translations
- Language detection and persistence
- Locale-aware formatting (dates, numbers, currency)
- Dynamic language switching
- Regional locale support (en-GB, en-US, es-ES, es-MX, etc.)

---

## Architecture Overview

```
/apps/web/src/i18n/
├── config.ts              # Main i18next configuration
├── languages.ts            # Supported languages & locale definitions
├── formatters.ts           # Locale-aware formatting utilities
└── locales/
    ├── en-GB.json         # English (UK) translations
    ├── es-ES.json         # Spanish (Spain) translations
    └── [future locales]   # Additional language files
```

**Key Components:**
- **LanguageSwitcher**: UI component for language selection
- **useLocale**: Hook for locale-aware formatting
- **useTranslation**: React i18next hook for translations

---

## Adding a New Language

### Step 1: Create Translation File

Create a new JSON file in `/apps/web/src/i18n/locales/` using the BCP-47 locale code:

```bash
# Example: Adding French (France)
cp apps/web/src/i18n/locales/en-GB.json apps/web/src/i18n/locales/fr-FR.json
```

**File naming convention:**
- `en-GB.json` - English (United Kingdom)
- `es-ES.json` - Spanish (Spain)
- `fr-FR.json` - French (France)
- `fr-CA.json` - French (Canada)
- `de-DE.json` - German (Germany)

### Step 2: Update Language Configuration

Add the new language to `/apps/web/src/i18n/languages.ts`:

```typescript
export const SUPPORTED_LANGUAGES: Language[] = [
  // ... existing languages
  {
    code: 'fr-FR',
    languageCode: 'fr',
    name: 'French (France)',
    nativeName: 'Français (France)',
    region: 'France',
    flag: '🇫🇷',
    currency: 'EUR',
    dateFormat: 'dd/MM/yyyy',
    numberFormat: { decimal: ',', thousands: ' ' },
  },
];
```

**Language Object Properties:**
- `code`: Full BCP-47 locale code (e.g., 'fr-FR')
- `languageCode`: Base language code (e.g., 'fr')
- `name`: English display name
- `nativeName`: Native language display name
- `region`: Geographic region name
- `flag`: Unicode flag emoji
- `currency`: Default currency code (ISO 4217)
- `dateFormat`: Preferred date format pattern
- `numberFormat`: Number formatting preferences
- `rtl`: Set to `true` for right-to-left languages (optional)

### Step 3: Update i18n Configuration

Add the new language resources to `/apps/web/src/i18n/config.ts`:

```typescript
import frFRTranslations from './locales/fr-FR.json';

const resources = {
  'en-GB': { translation: enGBTranslations },
  'es-ES': { translation: esESTranslations },
  'fr-FR': { translation: frFRTranslations }, // Add new import
};
```

### Step 4: Translate Content

Translate all keys in the new JSON file. The structure follows this pattern:

```json
{
  "common": {
    "loading": "Chargement...",
    "save": "Sauvegarder",
    "cancel": "Annuler"
  },
  "navigation": {
    "dashboard": "Tableau de bord",
    "announcements": "Annonces"
  },
  "dashboard": {
    "title": "Tableau de bord communautaire",
    "stats": {
      "totalUnits": "Total des unités"
    }
  }
}
```

---

## Language Detection & Persistence

The system follows a simple, predictable language detection flow:

### Detection Flow
1. **localStorage** 🗄️ - Previously selected language (if valid and supported)
2. **Browser locale** 🌍 - `navigator.language` (if exactly supported)
3. **Base language match** 🔄 - Base language fallback (e.g., `en-US` → `en-GB`)
4. **Default fallback** ⚡ - `en-GB` (configurable)

### Examples
- **First visit with Spanish browser**: `navigator.language = 'es-ES'` → Uses `es-ES` → Persists to localStorage
- **Return visit**: localStorage has `es-ES` → Uses `es-ES`
- **US English browser**: `navigator.language = 'en-US'` → No exact match → Finds `en-GB` → Uses `en-GB`
- **Unsupported browser**: `navigator.language = 'fr-FR'` → No match → Uses default `en-GB`

### Implementation
```typescript
// Simple detection logic in /apps/web/src/i18n/config.ts
const getInitialLanguage = (): string => {
  // 1. localStorage → 2. browser → 3. base match → 4. default
}
```

### Persistence
- **All selections are automatically persisted** to localStorage
- **Manual language changes** via LanguageSwitcher are persisted
- **Survives page reloads, login/logout, and browser restarts**
- **User preferences** stored as `i18nextLng` in localStorage

---

## Translation Key Organization

### Key Structure
```json
{
  "common": {},          // Shared UI elements (buttons, labels)
  "navigation": {},      // Menu items, navigation
  "dashboard": {},       // Dashboard-specific content
  "announcements": {},   // Announcements page
  "incidents": {},       // Incidents page
  "documents": {},       // Documents page
  "meetings": {},        // Meetings page
  "settings": {},        // Settings & profile
  "auth": {},           // Authentication pages
  "activity": {}        // Activity feed
}
```

### Naming Conventions
- Use **camelCase** for keys: `totalUnits`, `activeIncidents`
- Group related keys: `stats.totalUnits`, `stats.activeIncidents`
- Use descriptive names: `noAnnouncementsFilter` vs `noItems`
- Include context: `loadError` vs just `error`

### Pluralization
React i18next supports automatic pluralization:

```json
{
  "hoursAgo": "{{count}} hour ago",
  "hoursAgo_plural": "{{count}} hours ago"
}
```

Usage:
```typescript
t('activity.hoursAgo', { count: 1 });  // "1 hour ago"
t('activity.hoursAgo', { count: 3 });  // "3 hours ago"
```

---

## Language Switcher Component

### Usage
```typescript
import { LanguageSwitcher } from '@/components/ui/LanguageSwitcher';

// Button variant (cycles through languages)
<LanguageSwitcher variant="button" size="md" />

// Dropdown variant (shows all languages in menu)
<LanguageSwitcher variant="dropdown" size="lg" />
```

### Props
- `variant`: `'button' | 'dropdown'` - UI style
- `size`: `'sm' | 'md' | 'lg'` - Component size
- `className`: Additional CSS classes

### When to Use Each Variant
- **Button variant**: 2 languages (current implementation)
- **Dropdown variant**: 3+ languages (recommended for expansion)

### Switching to Dropdown
When adding a third language, update the TopBar:

```typescript
// In /apps/web/src/components/dashboard/TopBar.tsx
<LanguageSwitcher variant="dropdown" size="md" />
```

---

## Locale-Aware Formatting

### Using the useLocale Hook

```typescript
import { useLocale } from '@/hooks/useLocale';

function MyComponent() {
  const {
    locale,           // Current locale (en-GB)
    language,         // Language object
    currency,         // Default currency (GBP)
    formatCurrency,   // Format currency amounts
    formatDate,       // Format dates
    formatNumber,     // Format numbers
    formatPercentage  // Format percentages
  } = useLocale();

  return (
    <div>
      <p>Price: {formatCurrency(1250.50)}</p>
      <p>Date: {formatDate(new Date())}</p>
      <p>Number: {formatNumber(1234567.89)}</p>
    </div>
  );
}
```

### Available Formatters

**Currency:**
```typescript
formatCurrency(amount: number, currencyCode?: string)
// Examples:
formatCurrency(1250.50)      // "£1,250.50" (en-GB)
formatCurrency(1250.50)      // "1.250,50 €" (es-ES)
formatCurrency(1250.50, 'USD') // "$1,250.50"
```

**Numbers:**
```typescript
formatNumber(value: number, options?: Intl.NumberFormatOptions)
// Examples:
formatNumber(1234.56)        // "1,234.56" (en-GB)
formatNumber(1234.56)        // "1.234,56" (es-ES)
```

**Dates:**
```typescript
formatDate(date: Date | string, options?: Intl.DateTimeFormatOptions)
formatDateTime(date: Date | string)
formatTime(date: Date | string)
formatRelativeTime(date: Date | string)
```

---

## Best Practices

### 1. Always Use Translation Functions
❌ **Don't:**
```typescript
<button>Save</button>
<h1>Dashboard</h1>
```

✅ **Do:**
```typescript
<button>{t('common.save')}</button>
<h1>{t('dashboard.title')}</h1>
```

### 2. Add Translation Dependencies to useMemo
```typescript
const stats = useMemo(() => [
  { key: 'totalUnits', value: 150, label: t('dashboard.stats.totalUnits') },
  { key: 'activeIncidents', value: 3, label: t('dashboard.stats.activeIncidents') }
], [dashboard, t]); // ← Include 't' in dependencies
```

### 3. Handle Loading States
```typescript
const { t } = useTranslation();

if (isLoading) {
  return <div>{t('common.loading')}</div>;
}
```

### 4. Use Interpolation for Dynamic Content
```json
{
  "welcomeMessage": "Welcome back, {{name}}!",
  "storageUsed": "{{used}} GB / {{total}} GB used"
}
```

```typescript
t('welcomeMessage', { name: user.name });
t('documents.storageUsed', { used: 2.5, total: 10 });
```

---

## Testing Translations

### 1. Visual Testing
- Switch languages using the LanguageSwitcher
- Verify all text updates correctly
- Check for layout issues with longer text
- Test RTL languages if supported

### 2. Missing Translation Detection
React i18next shows missing keys in development:
- Missing keys appear as the key path: `dashboard.stats.totalUnits`
- Check browser console for i18next warnings

### 3. Unit Testing
```typescript
import { render } from '@testing-library/react';
import { I18nextProvider } from 'react-i18next';
import i18n from '@/i18n/config';

const renderWithI18n = (component) => {
  return render(
    <I18nextProvider i18n={i18n}>
      {component}
    </I18nextProvider>
  );
};
```

---

## Common Issues & Solutions

### 1. Language Switcher Not Working
**Symptoms:** Clicking language switcher doesn't change language
**Solutions:**
- Check i18n config imports match actual file names
- Verify resources object maps to correct translation files
- Ensure localStorage permissions are available

### 2. Translations Show as Keys
**Symptoms:** See `dashboard.title` instead of actual text
**Solutions:**
- Check translation file exists and is properly formatted JSON
- Verify key exists in translation file
- Check i18n config imports the file correctly

### 3. Layout Breaking with New Language
**Solutions:**
- Test with longer text (German, Dutch)
- Use CSS `text-overflow: ellipsis` for constrained spaces
- Consider responsive design for varying text lengths

### 4. Numbers/Dates Not Formatting
**Solutions:**
- Use `useLocale` hook instead of hardcoded formatting
- Ensure locale data is available in browser
- Check regional format definitions in `languages.ts`

### 5. Language Reverting to Default After Login/Reload
**Symptoms:** Language switches back to en-GB after page reload or login
**Solutions:**
- Check browser console for language detection logs (🌐 prefix)
- Verify localStorage contains `i18nextLng` with valid language code
- Clear localStorage and test detection flow: `localStorage.removeItem('i18nextLng')`
- Ensure selected language exists in `SUPPORTED_LANGUAGES` array

**Debug steps:**
```javascript
// Check current stored language
localStorage.getItem('i18nextLng')

// Check supported languages
console.log(getSupportedLanguageCodes())

// Force language detection
localStorage.removeItem('i18nextLng')
location.reload()
```

---

## Performance Considerations

### 1. Lazy Loading Languages
For many languages, consider lazy loading translation files:

```typescript
// Future enhancement - not implemented yet
const loadTranslation = async (locale: string) => {
  const translation = await import(`./locales/${locale}.json`);
  return translation.default;
};
```

### 2. Translation File Size
- Keep translation files under 100KB
- Split large files by feature if needed
- Remove unused keys during builds

### 3. Caching
- Translation files are cached by browser
- localStorage persists language preference
- No server requests needed for static translations

---

## Integration with Backend Translation

This frontend i18n system handles **static UI translations**. For **dynamic content translation** (announcements, documents, user-generated content), refer to the backend translation architecture:

- Static UI: Frontend i18next (this system)
- Dynamic content: Backend TranslationService via API
- Combined: User sees UI in preferred language + auto-translated content

---

## Future Enhancements

### Phase 1 Roadmap
- [x] Basic en-GB/es-ES support
- [x] LanguageSwitcher component
- [x] Locale-aware formatting
- [ ] Dropdown variant for 3+ languages

### Phase 2 Ideas
- [ ] Lazy loading for large language sets
- [ ] Translation memory integration
- [ ] Admin interface for translation management
- [ ] Automated translation validation
- [ ] Context-aware translations

---

## Quick Reference

### Add New Language Checklist
1. [ ] Create translation file: `locales/{locale}.json`
2. [ ] Copy from existing file and translate all keys
3. [ ] Add language object to `languages.ts`
4. [ ] Import and add to resources in `config.ts`
5. [ ] Test language switching functionality
6. [ ] Verify formatting (dates, numbers, currency)
7. [ ] Check for layout issues with new text

### Language Detection Flow Debug
```javascript
// Check detection flow in browser console
localStorage.getItem('i18nextLng')        // Step 1: localStorage
navigator.language                       // Step 2: browser locale
getSupportedLanguageCodes()             // Available languages
```

**Console logs to look for:**
- 🌐 Using stored language: es-ES
- 🌐 Using browser locale: en-GB
- 🌐 Using matching base language: en-GB for browser: en-US
- 🌐 Using default language: en-GB

### File Locations
- **Configuration:** `/apps/web/src/i18n/config.ts`
- **Languages:** `/apps/web/src/i18n/languages.ts`
- **Translations:** `/apps/web/src/i18n/locales/*.json`
- **Component:** `/apps/web/src/components/ui/LanguageSwitcher.tsx`
- **Hook:** `/apps/web/src/hooks/useLocale.ts`
- **Formatters:** `/apps/web/src/i18n/formatters.ts`

