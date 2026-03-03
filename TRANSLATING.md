# Translating Mezo Documentation

Thank you for your interest in translating the Mezo docs! This guide explains how internationalization (i18n) works in this project and how you can contribute translations.

## How i18n works

This site is built with [Astro Starlight](https://starlight.astro.build/), which has built-in i18n support. Key concepts:

- **English is the root locale.** English source files live directly in `src/content/docs/` with no language prefix.
- **Other languages use subdirectories.** Translated files go in `src/content/docs/<lang>/`, mirroring the English file structure. For example, the Spanish translation of `src/content/docs/docs/users/musd.mdx` lives at `src/content/docs/es/docs/users/musd.mdx`.
- **Two config files** register each locale: `astro.config.mjs` (Starlight) and `lunaria.config.json` (translation tracking dashboard).

## How to add a new language

1. **`astro.config.mjs`** — add an entry to the `locales` object inside the Starlight config:

   ```js
   locales: {
     root: { label: 'English', lang: 'en' },
     es: { label: 'Español', lang: 'es' },
     fr: { label: 'Français', lang: 'fr' },
     // Add your language here
   },
   ```

2. **`lunaria.config.json`** — add an entry to the `locales` array:

   ```json
   {
     "label": "Français",
     "lang": "fr"
   }
   ```

3. **Create the content directory** at `src/content/docs/<lang>/` (e.g., `src/content/docs/fr/`).

## How to translate a page

1. Find the English source file you want to translate (e.g., `src/content/docs/docs/users/musd.mdx`).
2. Copy it to the corresponding path under your language directory (e.g., `src/content/docs/fr/docs/users/musd.mdx`).
3. Translate the content, keeping the frontmatter structure intact. Update the `title` and `description` fields but leave `slug` and other technical fields unchanged.
4. Preserve all links, images, and component imports as-is — only translate the human-readable text.

## File structure conventions

```
src/content/docs/
├── docs/                    # English (root locale)
│   ├── users/
│   │   ├── musd.mdx
│   │   └── ...
│   └── developers/
│       └── ...
├── es/                      # Spanish
│   └── docs/
│       └── users/
│           └── musd.mdx
└── fr/                      # French
    └── docs/
        └── users/
            └── musd.mdx
```

## Local preview

Run the dev server to preview your translations:

```bash
npm run dev
```

Translated pages are available at `http://localhost:4321/docs/<lang>/...` (e.g., `/docs/fr/docs/users/musd`).

## Lunaria dashboard

The [Lunaria dashboard](https://mezo.org/docs/lunaria/) tracks translation progress across all locales. It automatically detects which pages have been translated and which are outdated based on git history.

After your translation is merged, the dashboard updates on the next build.
