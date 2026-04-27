# Composable Page Layout (AI-Designed Storefront)

Design and customize your shop's page layout using AI. The composable system lets you define the structure, sections, and visual style of your storefront through a JSON configuration — no coding required.

> **Prerequisite:** The shop must have at least one section created (see [sections.md](sections.md)) before the composable layout can display products.

---

## How It Works

1. **Page Config** — A JSON tree that describes the page structure (header, sections, footer, banners)
2. **Components** — Building blocks like `Section.Grid`, `Section.Slider`, `Section.HeroBanner`, `Container.Root`
3. **AI Editor** — Describe what you want in plain language, and the AI generates the config JSON
4. **Save & Apply** — The saved config is stored on the shop and rendered live on the storefront

> **When no page config is set**, the shop uses the legacy layout (sections rendered in database order). Setting a page config activates the composable renderer.

---

## Get Current Page Config

```graphql
query ShopPageConfig($shopId: String!) {
  shopId(shopId: $shopId) {
    pageConfig
  }
}
```

Returns `null` if no composable layout is set (shop uses legacy mode), or a JSON object if active.

---

## Set Page Config

```graphql
mutation UpdatePageConfig($shopId: String!, $pageConfig: JSON!) {
  updatePageConfig(shopId: $shopId, pageConfig: $pageConfig)
}
```

Returns `true` on success.

### ⚠️ CONFIRMATION REQUIRED
Before saving, show the user a summary of what the layout will look like. Wait for explicit "Yes" before calling the mutation.

---

## Clear Page Config (Revert to Legacy)

To revert to the default/legacy layout, set `pageConfig` to `null`:

```graphql
mutation ClearPageConfig($shopId: String!) {
  updatePageConfig(shopId: $shopId, pageConfig: null)
}
```

---

## Page Config JSON Structure

```json
{
  "version": 1,
  "root": {
    "id": "root-shell",
    "type": "Layout.Shell",
    "children": [
      { "id": "header", "type": "Layout.Header" },
      {
        "id": "sections-container",
        "type": "Container.Root",
        "props": { "className": "max-w-7xl mx-auto min-h-dvh h-full" },
        "children": [
          {
            "id": "sections-fragment",
            "type": "fragment",
            "children": []
          }
        ]
      },
      { "id": "footer", "type": "Layout.Footer" }
    ]
  }
}
```

### Rules

| Rule | Description |
|---|---|
| `version` | Must always be `1` |
| Root node | Must be `Layout.Shell` |
| Every node | Must have a unique `id` string |
| `Layout.Header` | Should be first child of Shell |
| `Layout.Footer` | Should be last child of Shell |
| `sections-fragment` | A `fragment` node inside `Container.Root` — this is where the shop's existing sections (from the sections system) are injected. **DO NOT remove or rename this node.** |
| `fragment` type | Groups children with no DOM wrapper (invisible grouping) |

---

## Available Components

### Layout Components

| Type | Description | Props | Children? |
|---|---|---|---|
| `Layout.Shell` | Root wrapper — must be the tree root | none | ✅ Yes |
| `Layout.Header` | Shop header/navbar | none | ❌ No |
| `Layout.Footer` | Shop footer | none | ❌ No |
| `Layout.MobileNav` | Mobile bottom navigation bar | none | ❌ No |

### Container Components

| Type | Description | Props | Children? |
|---|---|---|---|
| `Container.Root` | Generic `<div>` wrapper | `className` (string) | ✅ Yes |
| `fragment` | Invisible grouping — no DOM output | none | ✅ Yes |

### Section Components

| Type | Description | Props | Children? |
|---|---|---|---|
| `Section.Wrapper` | Section wrapper with title bar | `sectionId` (string), `title` (string), `showTitle` (bool), `titleAlignment` ("LEFT"/"CENTER"/"RIGHT"), `padding` ("NONE"/"SM"/"MD"/"LG") | ✅ Yes |
| `Section.Slider` | Product slider/carousel | `sectionId` (string), `shopId` (string), `locale` (string), `priority` (bool) | ❌ No |
| `Section.Grid` | Product grid | `sectionId` (string), `shopId` (string), `locale` (string), `priority` (bool) | ❌ No |
| `Section.Horizontal` | Horizontal scroll product list | `sectionId` (string), `shopId` (string), `locale` (string), `priority` (bool) | ❌ No |
| `Section.HeroBanner` | Image banner slider | `images` (string[]), `autoPlayInterval` (number), `showControls` (bool), `showIndicators` (bool), `showCounter` (bool), `className` (string) | ❌ No |

### Product Card Components

| Type | Description | Props | Children? |
|---|---|---|---|
| `ProductCard.Root` | Product card (not yet used as a tree node — card style is controlled via theme) | `productId` (string) | ❌ No |

---

## Working with Sections

Section components (`Section.Grid`, `Section.Slider`, etc.) reference existing sections by `sectionId`. These sections must already exist in the database (created via `createSection` — see [sections.md](sections.md)).

### Important: `sectionId` is Required

Each section component **must** have a valid `sectionId` that matches a section in the database. The section's `dataSource` and `filterCondition` determine which products are shown.

### Fetching Section IDs

Before building a page config, fetch the shop's sections:

```graphql
query PageSections($shopId: String!, $page: String!) {
  getPageSections(shopId: $shopId, page: $page) {
    id name page displayOrder dataSource layout
  }
}
```

Use the returned `id` values as `sectionId` props in the page config.

---

## AI-Generated Layouts

When a user describes their desired layout in natural language, construct the JSON config that matches their request. Here are common patterns:

### Example 1: Default Layout (3 sections)

A clean layout with header, a grid of featured products, a slider of new arrivals, and a footer.

```json
{
  "version": 1,
  "root": {
    "id": "root-shell",
    "type": "Layout.Shell",
    "children": [
      { "id": "header", "type": "Layout.Header" },
      {
        "id": "hero-banner",
        "type": "Section.HeroBanner",
        "props": {
          "images": ["https://cdn.riverbase.org/your-banner.webp"],
          "autoPlayInterval": 5000,
          "showControls": true,
          "showIndicators": true,
          "showCounter": false,
          "className": "w-full"
        }
      },
      {
        "id": "sections-container",
        "type": "Container.Root",
        "props": { "className": "max-w-7xl mx-auto" },
        "children": [
          {
            "id": "featured-grid",
            "type": "Section.Grid",
            "props": { "sectionId": "<SECTION_ID_1>", "shopId": "<SHOP_ID>", "locale": "en", "priority": true }
          },
          {
            "id": "new-arrivals-slider",
            "type": "Section.Slider",
            "props": { "sectionId": "<SECTION_ID_2>", "shopId": "<SHOP_ID>", "locale": "en", "priority": true }
          },
          {
            "id": "sections-fragment",
            "type": "fragment",
            "children": []
          }
        ]
      },
      { "id": "footer", "type": "Layout.Footer" }
    ]
  }
}
```

### Example 2: Hero Banner + Grid + Horizontal Scroll

A visually rich layout with a full-width banner, a product grid, and a horizontal scroll section.

```json
{
  "version": 1,
  "root": {
    "id": "root-shell",
    "type": "Layout.Shell",
    "children": [
      { "id": "header", "type": "Layout.Header" },
      {
        "id": "hero",
        "type": "Section.HeroBanner",
        "props": {
          "images": [
            "https://cdn.riverbase.org/banner1.webp",
            "https://cdn.riverbase.org/banner2.webp"
          ],
          "autoPlayInterval": 4000,
          "showControls": true,
          "showIndicators": true,
          "showCounter": true,
          "className": ""
        }
      },
      {
        "id": "content-area",
        "type": "Container.Root",
        "props": { "className": "max-w-7xl mx-auto space-y-8" },
        "children": [
          {
            "id": "all-products",
            "type": "Section.Wrapper",
            "props": { "sectionId": "<SECTION_ID_1>", "title": "All Products", "showTitle": true, "titleAlignment": "CENTER", "padding": "LG" },
            "children": [
              {
                "id": "products-grid",
                "type": "Section.Grid",
                "props": { "sectionId": "<SECTION_ID_1>", "shopId": "<SHOP_ID>", "locale": "en", "priority": true }
              }
            ]
          },
          {
            "id": "trending",
            "type": "Section.Wrapper",
            "props": { "sectionId": "<SECTION_ID_2>", "title": "Trending Now", "showTitle": true, "titleAlignment": "LEFT", "padding": "MD" },
            "children": [
              {
                "id": "trending-scroll",
                "type": "Section.Horizontal",
                "props": { "sectionId": "<SECTION_ID_2>", "shopId": "<SHOP_ID>", "locale": "en", "priority": false }
              }
            ]
          },
          {
            "id": "sections-fragment",
            "type": "fragment",
            "children": []
          }
        ]
      },
      { "id": "footer", "type": "Layout.Footer" }
    ]
  }
}
```

### Example 3: Minimal Layout (Just Header + Sections + Footer)

The simplest composable layout — header, server-rendered sections, footer.

```json
{
  "version": 1,
  "root": {
    "id": "root-shell",
    "type": "Layout.Shell",
    "children": [
      { "id": "header", "type": "Layout.Header" },
      {
        "id": "sections-container",
        "type": "Container.Root",
        "props": { "className": "max-w-7xl mx-auto min-h-dvh h-full" },
        "children": [
          {
            "id": "sections-fragment",
            "type": "fragment",
            "children": []
          }
        ]
      },
      { "id": "footer", "type": "Layout.Footer" }
    ]
  }
}
```

### Example 4: Multi-Banner Promotional Layout

A promotional layout with multiple banners interleaved with product sections.

```json
{
  "version": 1,
  "root": {
    "id": "root-shell",
    "type": "Layout.Shell",
    "children": [
      { "id": "header", "type": "Layout.Header" },
      {
        "id": "main-banner",
        "type": "Section.HeroBanner",
        "props": {
          "images": ["https://cdn.riverbase.org/promo-main.webp"],
          "autoPlayInterval": 0,
          "showControls": false,
          "showIndicators": false,
          "showCounter": false,
          "className": ""
        }
      },
      {
        "id": "content",
        "type": "Container.Root",
        "props": { "className": "max-w-7xl mx-auto space-y-6" },
        "children": [
          {
            "id": "featured",
            "type": "Section.Grid",
            "props": { "sectionId": "<SECTION_ID_1>", "shopId": "<SHOP_ID>", "locale": "en", "priority": true }
          },
          {
            "id": "mid-banner",
            "type": "Section.HeroBanner",
            "props": {
              "images": ["https://cdn.riverbase.org/promo-mid.webp"],
              "autoPlayInterval": 0,
              "showControls": false,
              "showIndicators": false,
              "showCounter": false,
              "className": ""
            }
          },
          {
            "id": "new-arrivals",
            "type": "Section.Slider",
            "props": { "sectionId": "<SECTION_ID_2>", "shopId": "<SHOP_ID>", "locale": "en", "priority": false }
          },
          {
            "id": "sections-fragment",
            "type": "fragment",
            "children": []
          }
        ]
      },
      { "id": "footer", "type": "Layout.Footer" }
    ]
  }
}
```

---

## Common User Requests & How to Handle

| User Says | Action |
|---|---|
| "Make my store look modern" | Generate a layout with hero banner + centered section titles + spacious padding |
| "Add a banner slider at the top" | Add a `Section.HeroBanner` as the first child after `Layout.Header` |
| "Show products in a grid" | Use `Section.Grid` with the appropriate `sectionId` |
| "Add a carousel of products" | Use `Section.Slider` with the appropriate `sectionId` |
| "I want a horizontal scroll section" | Use `Section.Horizontal` with the appropriate `sectionId` |
| "Reset my layout" | Set `pageConfig` to `null` (reverts to legacy mode) |
| "What does my layout look like?" | Fetch `pageConfig` and describe the structure to the user |
| "Remove the banner" | Remove the `Section.HeroBanner` node from the tree |
| "Put a banner between two product sections" | Insert `Section.HeroBanner` between the section nodes |
| "Change the section title to 'New Arrivals'" | Update the `title` prop on the `Section.Wrapper` node |
| "Make the title centered" | Set `titleAlignment` to `"CENTER"` on the `Section.Wrapper` |
| "Add more spacing between sections" | Add `"space-y-8"` or `"space-y-12"` to the `Container.Root` className |
| "Full-width banner" | Place `Section.HeroBanner` outside of `Container.Root` (directly under `Layout.Shell`) |

---

## Workflow: Setting Up a Composable Layout

1. **Check existing sections** — Run `getPageSections` to see what sections the shop has.
2. **Fetch current page config** — Run `ShopPageConfig` to see if a config already exists.
3. **Build or modify the config** — Based on the user's request, construct the JSON tree.
4. **Show preview** — Describe the layout to the user in plain language.
5. **Get confirmation** — Ask "Ready to apply? Reply Yes to save."
6. **Save** — Call `updatePageConfig` with the JSON.
7. **Verify** — Re-fetch `ShopPageConfig` to confirm it was saved.

### Example Conversation Flow

**User:** "I want a modern layout with a big banner at the top, then my featured products in a grid, and new arrivals in a slider"

**Agent:**
1. Fetches sections: `getPageSections(shopId, "home")` → gets section IDs
2. Builds the config JSON (Example 2 from above, with real section IDs)
3. Shows the user: "Here's what I'll set up:
   - 🔝 Full-width image banner at the top
   - 📦 'All Products' in a grid layout
   - 🔄 'New Arrivals' in a horizontal slider
   - Footer at the bottom
   
   Ready to apply? Reply Yes to save."
4. On "Yes", calls `updatePageConfig`

---

## Workflow: Modifying an Existing Layout

1. **Fetch current config** — `ShopPageConfig` to get the existing JSON tree.
2. **Modify the JSON** — Add/remove/reorder nodes based on the request.
3. **Show what changed** — Describe the modification to the user.
4. **Get confirmation** → **Save** → **Verify**.

### Important: Preserve Existing Node IDs

When modifying a config, keep the existing `id` values for unchanged nodes. Only add new unique IDs for new nodes.

---

## Agent Notes

- **Images in HeroBanner** must be uploaded CDN URLs (use `POST /uploads/s3`). Don't use placeholder URLs.
- **Section IDs are real database IDs** — they come from the sections system, not made up. Always fetch with `getPageSections` first.
- **`sections-fragment` is sacred** — it's the injection point for server-rendered sections. Removing it breaks the layout. Always include it.
- **`locale` should match the shop's locale** — typically `"en"` for English-language shops. Check the shop's settings if unsure.
- **`priority: true`** on sections means the first few product images will use higher priority loading (LCP optimization).
- **Container className uses Tailwind CSS** — you can use any Tailwind utility: `max-w-7xl mx-auto space-y-8 p-4 bg-gray-50 rounded-xl`, etc.
- **The composable system is additive** — it doesn't replace the sections system. Sections still live in the database and are referenced by ID. The page config just controls their arrangement and wrapping.
- **No `Layout.Header`/`Layout.Footer`/`Layout.MobileNav` content is rendered** — these are no-ops in the composable renderer because the platform app layout (LayoutShop) already provides the real header/footer/mobile nav. They exist in the tree for future theming support.
- **Themes** — Visual style (card appearance, colors, spacing) is controlled by the theme system (see [appearance.md](appearance.md)). The page config controls structure/arrangement, not style.
