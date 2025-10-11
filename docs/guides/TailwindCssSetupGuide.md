# Tailwind CSS Setup Guide

This guide explains how to get Tailwind CSS 4 running in modern front-end stacks without relying on project-specific tooling. It focuses on two common scenarios you will encounter in this repository and elsewhere:

1. **Astro + React islands**
2. **React + TypeScript (Vite or similar bundlers)**

Before diving into framework workflows, you will find shared concepts that apply regardless of the host framework.

---

## 1. Prerequisites & Installation Basics

- **Node.js 20+ and npm 9+** (check with `node -v && npm -v`)
- Tailwind 4 is bundled via `tailwindcss` and `@tailwindcss/vite`. Installing project dependencies (`npm install`) is enough; there is no `tailwind.config.js` to maintain unless you opt into optional authoring features.

General installation flow:

```bash
# From the project root
npm install

# (Optional) If you keep multiple apps in a monorepo, install per workspace
npm --prefix apps/admin-frontend install
```

Tailwind 4 uses the Vite plugin to detect all relevant files automatically. There is no `content` array or manual glob configuration.

---

## 2. Shared Tailwind Concepts

### Single import entry point

Tailwind 4 replaces the legacy trio of `@tailwind base/components/utilities` with a single directive:

```css
@import "tailwindcss";
```

You place this import in the stylesheet that each app consumes (for example, `src/styles/global.css` or `src/index.css`).

### Design tokens through CSS variables

Instead of extending Tailwind’s palette via configuration, define reusable values with CSS variables. This keeps tokens portable across frameworks:

```css
:root {
  color-scheme: light;
  --bg: #f5f6ff;
  --surface: #ffffff;
  --accent: #6366f1;
  --text: #101636;
}

:root[data-theme='dark'],
[data-theme='dark'] {
  color-scheme: dark;
  --bg: #090b12;
  --surface: #111827;
  --accent: #8b5cf6;
  --text: #f8fafc;
}
```

Use Tailwind’s arbitrary value syntax to reference these tokens in markup:

```html
<div class="bg-[color:var(--surface)] text-[color:var(--text)]">
  …
</div>
```

### Theme switching strategy

1. Store tokens in light/dark blocks as shown above.
2. Toggle a `data-theme` attribute on the `<html>` element.
3. Use a script or component to read system preferences, persist the chosen theme, and update `document.documentElement.dataset.theme`.

This approach works the same way in Astro or pure React and avoids Tailwind’s `dark:` modifier entirely.

---

## 3. Astro + Tailwind CSS 4

Astro uses Vite under the hood, so you register Tailwind via the Vite plugin in `astro.config.mjs`:

```js
// astro.config.mjs
import { defineConfig } from 'astro/config';
import react from '@astrojs/react';
import tailwindcss from '@tailwindcss/vite';

export default defineConfig({
  integrations: [react()],
  vite: {
    plugins: [tailwindcss()],
    server: {
      proxy: {
        '/api': 'http://localhost:5143'
      }
    }
  }
});
```

Key points for any Astro project:

- Import a global stylesheet in your base layout (e.g., `import '../styles/global.css';`).
- Ensure that stylesheet contains `@import "tailwindcss";` and any tokens/utilities you rely on.
- For theme initialization, add an inline script before paint in your layout to apply the stored or preferred theme:

```html
<script is:inline>
  (function() {
    var stored = localStorage.getItem('theme');
    var prefersDark = window.matchMedia('(prefers-color-scheme: dark)').matches;
    var theme = stored || (prefersDark ? 'dark' : 'light');
    document.documentElement.dataset.theme = theme;
  })();
</script>
```

Once wired up, `npm run dev` gives hot-reload with Tailwind utilities available inside `.astro`, `.tsx`, and plain `.ts` files.

---

## 4. React + TypeScript (Vite or similar bundlers)

For a standalone React app (Vite, Next.js with the Vite plugin, etc.), register Tailwind in the bundler config and import the shared stylesheet at the top of your entry file.

Example with Vite:

```ts
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import tailwindcss from '@tailwindcss/vite';
import path from 'node:path';

export default defineConfig({
  plugins: [react(), tailwindcss()],
  resolve: {
    alias: {
      '@styles': path.resolve(__dirname, '../src/styles')
    }
  }
});
```

In your entry stylesheet (`src/index.css` or `src/styles/global.css`):

```css
@import "@styles/global.css"; /* re-export the shared Tailwind + tokens file */

html,
body,
#root {
  height: 100%;
}
```

Then, in `main.tsx` (or `main.jsx`):

```tsx
import './index.css';
import App from './App';
import React from 'react';
import ReactDOM from 'react-dom/client';

ReactDOM.createRoot(document.getElementById('root')!).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);
```

Theme toggling can live in a reusable hook/component:

```tsx
const THEMES = ['light', 'dark'] as const;

const applyTheme = (theme: Theme) => {
  document.documentElement.dataset.theme = theme;
  localStorage.setItem('theme', theme);
};
```

Attach that logic to a button, context provider, or any UI element you prefer.

---

## 5. Common Tasks for Junior Developers

- **Build new UI:** Use Tailwind utilities directly in Astro components or React JSX. Reach for layout classes (`flex`, `grid`, `gap-4`) and pair them with token-based colors (`bg-[color:var(--surface-muted)]`).
- **Add a design token:** Define it under both light and dark blocks in the shared stylesheet; consume it with arbitrary values. No configuration files need updating.
- **Share tokens between apps:** Import the same `global.css` (or similar) file in each workspace. Aliases like `@styles` simplify cross-app imports.
- **Create bespoke utilities:** Tailwind 4 supports authoring via `@utility` and `@layer` directly in CSS. Use these sparingly once you identify repeated patterns.

---

## 6. Testing & Debugging Tips

- Start the relevant dev server (`npm run dev`, `npm run admin:dev`, etc.). Tailwind updates render instantly via hot module replacement.
- Inspect the `<html>` element to confirm the `data-theme` attribute updates when the user toggles themes.
- If a class does not compile, check that the literal string exists in source files; Tailwind’s compiler needs to see the actual class name (avoid runtime concatenation).
- For new stylesheets, make sure they are imported somewhere reachable by the bundler or the utilities won’t be generated.

---

## 7. External Resources

- [Tailwind CSS v4 documentation](https://tailwindcss.com/docs) — reference for zero-config setup and new authoring features.
- [Arbitrary value reference](https://tailwindcss.com/docs/adding-custom-styles#using-arbitrary-values) — explains how to plug CSS variables into utility classes.
- [Astro + Tailwind docs](https://docs.astro.build/en/guides/integrations-guide/tailwind/) — Astro-specific notes if you need additional features.
- [Vite + Tailwind guide](https://tailwindcss.com/docs/guides/vite) — applies to any React/TypeScript project using Vite.

Use this guide as a blueprint for adding Tailwind to new apps or onboarding teammates. The same principles—single import, shared tokens, and attribute-driven theming—work across frameworks and keep your design system consistent.
