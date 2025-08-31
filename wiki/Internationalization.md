# Internationalization

The IBCOL platform features a comprehensive internationalization (i18n) system that supports multiple languages. This document explains how the translation system works and how to add or modify translations.

## Supported Languages

The platform supports over 20 languages, including:

- English (multiple variants)
- Chinese (Traditional and Simplified)
- Russian
- Ukrainian
- Portuguese
- Spanish
- French
- German
- Arabic
- And many more

Each language is identified by a locale code (e.g., `en-us`, `zh-hk`, `ru-ru`).

## Translation System Architecture

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│                 │     │                 │     │                 │
│  Translation    │────▶│  Translation    │────▶│  Component      │
│  Files (.json)  │     │  Helper         │     │  Rendering      │
│                 │     │                 │     │                 │
└─────────────────┘     └─────────────────┘     └─────────────────┘
```

### Key Components

1. **Translation Files**: JSON files organized by locale and page/section
2. **Translation Helper**: Utility functions for accessing translations
3. **Locale Detection**: System for detecting and setting user locale
4. **Language Selector**: UI component for changing language

## Translation File Structure

Translations are organized in the `/translations` directory with the following structure:

```
translations/
├── _default/
│   ├── _countries.json
│   ├── _global.json
│   ├── _project-categories.json
│   ├── _sectors.json
│   ├── about.json
│   ├── admin.json
│   ├── ambassadors.json
│   ├── contact.json
│   ├── home.json
│   ├── how.json
│   ├── index.js
│   ├── registration.json
│   ├── schedule.json
│   └── sponsors.json
├── en-us/
│   └── [same structure as _default]
├── zh-hk/
│   └── [same structure as _default]
└── [other locales]/
    └── [same structure as _default]
```

Each locale directory contains:

1. **Page-specific translations**: Files like `home.json`, `registration.json`
2. **Global translations**: Files like `_global.json`, `_countries.json`
3. **Index file**: `index.js` that exports all translations for the locale

## Translation File Example

Here's an example of a translation file (`home.json` for English):

```json
{
  "pageTitle": "International Blockchain Olympiad",
  "seoDescription": "International Blockchain Olympiad is a multidisciplinary design and building competition...",
  "mainHeading": "Help bring the world onto the Blockchain",
  "subHeading": "Meet and compete with the world's best blockchain builders",
  "register": "Registration",
  "howToJoin": "How to join",
  "competitionRules": "Competition Process",
  "scroll": "Scroll"
}
```

## Using Translations in Components

Translations are accessed using the `translate` helper function:

```jsx
import { translate } from 'helpers/translate.js';

// In a component:
const MyComponent = ({ locale }) => {
  // Get a translation from the 'home' namespace
  const pageTitle = translate('pageTitle', 'home', locale);
  
  return <h1>{pageTitle}</h1>;
};

// In a class component:
class MyClassComponent extends React.Component {
  translate = (key) => translate(key, 'home', this.props.locale);
  
  render() {
    return <h1>{this.translate('pageTitle')}</h1>;
  }
}
```

## Translation Helper

The `translate` function in `helpers/translate.js` handles:

1. Looking up translations by key and namespace
2. Falling back to default locale if translation is missing
3. Supporting nested translation keys (e.g., `section1.title`)

```javascript
export const translate = (key, namespace, locale) => {
  // Get the translation or fall back to default
  const translation = getTranslation(key, namespace, locale);
  
  // Return the translation or the key if not found
  return translation || key;
};
```

## Locale Detection and Switching

The application detects the user's locale through:

1. URL parameters (e.g., `/en-us/home`)
2. Browser language settings (as fallback)
3. Default locale (`en-us`) if no other locale is detected

Users can switch languages using the `LanguageSelectorComponent`, which:

1. Shows available languages
2. Allows selecting a new language
3. Redirects to the same page in the new locale

## Adding a New Language

To add a new language:

1. Create a new directory in `/translations` with the locale code (e.g., `de-de`)
2. Copy the structure from `_default` or another locale
3. Translate all JSON files
4. Add the locale to the supported locales list in `translations/index.js`

## Translation Mapping

The `translationsMapping.json` file maps locale codes to language names and flags:

```json
{
  "en-us": {
    "name": "English (US)",
    "flag": "us.svg"
  },
  "zh-hk": {
    "name": "繁體中文 (香港)",
    "flag": "hk.svg"
  }
}
```

## Best Practices

1. **Use Translation Keys**: Never hardcode text in components
2. **Organize by Page**: Keep translations organized by page/section
3. **Use Nested Keys**: For complex sections, use nested keys
4. **Include All Languages**: Ensure all supported languages have translations
5. **Test RTL Languages**: Test right-to-left languages like Arabic
6. **Use HTML in Translations**: For complex formatting, use HTML in translations with `dangerouslySetInnerHTML`

## HTML in Translations

Some translations contain HTML that is rendered using `dangerouslySetInnerHTML`:

```jsx
<div dangerouslySetInnerHTML={{ __html: translate('section1.htmlContent') }} />
```

Example translation with HTML:

```json
{
  "section1": {
    "htmlContent": "<p>This is <strong>bold</strong> text with a <a href='#'>link</a>.</p>"
  }
}
```

## Locale URL Structure

The application uses locale-prefixed URLs:

- `/en-us/home` - English (US) home page
- `/zh-hk/home` - Chinese (Hong Kong) home page
- `/ru-ru/home` - Russian home page

This is handled by the routing system, which includes locale as a parameter in all routes.

