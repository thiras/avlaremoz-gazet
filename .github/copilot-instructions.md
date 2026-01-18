# Copilot Instructions for Gazet Ghost Theme

## Project Overview
Gazet is a premium Ghost CMS theme (v5.118.0+) built with **TailwindCSS v4**, **Alpine.js**, and **Vite**. It uses Handlebars templating and supports multi-language localization.

## Architecture

### Template Hierarchy
- **`default.hbs`** - Base layout with head, body structure, Alpine.js initialization
- **`home.hbs`** - Homepage with configurable hero, topic sections, podcasts, events
- **`index.hbs`** - Main post listing (used for `/latest/` route)
- **`post.hbs`** - Single post with 5 layout variants controlled by `@custom.default_post_layout`
- **`custom-post-*.hbs`** - Template overrides for specific post layouts
- **`page-*.hbs`** - Special collection pages (events, podcasts, newsletters)

### Partials Organization
```
partials/
├── cards/           # Post card variants (vertical, horizontal, fullwidth)
├── sections/        # Homepage sections (hero, topics, writers, events)
├── banners/         # Header/footer/sidebar promotional banners
├── icons/           # SVG icon partials (use {{> icon name="..."}} helper)
└── *.hbs            # Core UI components (header, footer, menu, sidebar)
```

### Key Patterns

**Custom Settings** - Ghost Admin settings exposed via `@custom.*` in templates:
```handlebars
{{#match @custom.hero_posts_filter "featured"}}
  {{> sections/hero posts_filter="featured:true"}}
{{/match}}
```

**Content Filtering** - Use Ghost's `#get` helper with filter syntax:
```handlebars
{{#get "posts" filter="tag:-[hash-newsletter,hash-event]" include="tags,authors" limit="6"}}
```

**Hash Tags** - Internal tags prefixed with `hash-` (e.g., `hash-podcast`, `hash-event`) control content routing via `routes.yaml`.

## Development Workflow

```bash
npm run dev      # Start Vite dev server (port 2468) + watch mode
npm run build    # Production build to assets/built/
npm run test     # Validate with gscan (Ghost's official theme validator)
npm run zip      # Build + test + create distributable zip
```

**Vite proxies to Ghost** at `127.0.0.1:2368` - ensure Ghost is running locally.

## Deployment

Upload the theme zip via **Ghost Admin > Settings > Design > Change theme > Upload theme**. Always run `npm run zip` first to ensure validation passes.

## Styling

### TailwindCSS v4 Configuration
CSS entry point: [assets/src/css/main.css](assets/src/css/main.css)
- Uses `@theme` directive for design tokens (not `tailwind.config.js`)
- Custom properties: `--color-bgr`, `--color-typ`, `--color-brd`, `--color-brand`
- Typography plugin configured via `.prose` class tokens

### Color System
Colors are defined in `partials/config.hbs` as CSS variables from Ghost custom settings:
```css
--color-bgr: {{@custom.background_color}};
--color-typ: {{@custom.text_color}};
```

## JavaScript

Entry points in [vite.config.js](vite.config.js):
- `assets/src/js/index.js` - Main bundle (Alpine.js, utilities, Ghost API)
- `assets/src/js/swiper.js` - Carousel functionality (loaded separately)

Global functions exposed on `window`: `getPosts()`, `toggleColorScheme()`, `initDropdown()`

## Localization

Translation files in `locales/` (de, en, es, fr, it, nl, pr). Use the `{{t}}` helper:
```handlebars
{{t "Subscribe"}}
{{t "Page {current_page} of {total_pages}"}}
```

## Content Collections (routes.yaml)

| Route | Template | Filter |
|-------|----------|--------|
| `/newsletters/` | page-newsletters | `tag:[newsletter,hash-newsletter]` |
| `/events/` | page-events | `tag:[event,hash-event]` |
| `/podcasts/` | page-podcasts | `tag:[podcast,hash-podcast]` |
| `/latest/` | index | excludes hash-* tags |

## Card Components

Use appropriate card partial based on context:
- `{{> cards/post-card-vertical}}` - Grid layouts
- `{{> cards/post-card-horizontal}}` - List/feed layouts  
- `{{> cards/post-card-fullwidth}}` - Hero features
- Pass `card_class`, `media_class`, `title_class` for customization

## Testing Checklist

Before committing changes:
1. Run `npm run test` - must pass gscan validation
2. Test responsive layouts (mobile, tablet, desktop)
3. Verify Ghost Admin custom settings still work
4. Check localization keys if adding new UI text
