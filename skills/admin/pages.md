# Custom Pages

Create and manage custom content pages (About, Terms, Privacy, Contact, FAQ, etc.) for the shop's storefront.

> **Note**: The Pages RPC service must be running. If queries return "Failed to connect to RPC_PAGE", the service may be down.

## List All Pages

```graphql
query ListPages($shopId: String!) {
  getPages(shopId: $shopId) {
    id slug title status showInHeader showInFooter displayOrder coverImage
    seo { metaTitle metaDescription }
    createdAt updatedAt
  }
}
```

## List Published Pages Only

```graphql
query PublishedPages($shopId: String!) {
  getPublishedPages(shopId: $shopId) {
    id slug title status showInHeader showInFooter displayOrder coverImage
    seo { metaTitle metaDescription }
  }
}
```

## Get Page by Slug

```graphql
query PageBySlug($shopId: String!, $slug: String!) {
  getPageBySlug(shopId: $shopId, slug: $slug) {
    id slug title content status coverImage showInHeader showInFooter displayOrder
    seo { metaTitle metaDescription }
  }
}
```

## Get Page by ID

```graphql
query PageById($pageId: String!) {
  getPage(pageId: $pageId) {
    id slug title content status coverImage showInHeader showInFooter displayOrder
    seo { metaTitle metaDescription }
  }
}
```

## Create Page

```graphql
mutation CreatePage(
  $shopId: String!
  $slug: String!
  $title: String!
  $content: String!
  $coverImage: String
  $status: String!
  $showInHeader: Boolean
  $showInFooter: Boolean
  $displayOrder: Int
  $seo: String
) {
  createPage(
    shopId: $shopId
    slug: $slug
    title: $title
    content: $content
    coverImage: $coverImage
    status: $status
    showInHeader: $showInHeader
    showInFooter: $showInFooter
    displayOrder: $displayOrder
    seo: $seo
  ) { id slug title status }
}
```

### Parameters

| Field | Type | Required | Description |
|---|---|---|---|
| `slug` | String | ✅ | URL path: `"about"`, `"terms"`, `"contact"` — must be unique, lowercase, no spaces |
| `title` | String | ✅ | Page heading: `"About Us"`, `"Terms & Conditions"` |
| `content` | String | ✅ | Page body content (supports HTML) |
| `status` | String | ✅ | `"published"` or `"draft"` |
| `coverImage` | String | ❌ | Hero/banner image (CDN URL from `/uploads/s3`) |
| `showInHeader` | Boolean | ❌ | Show in header navigation dropdown |
| `showInFooter` | Boolean | ❌ | Show in footer links |
| `displayOrder` | Int | ❌ | Sort order for header/footer (lower = first) |
| `seo` | String | ❌ | JSON string: `{"metaTitle":"...", "metaDescription":"..."}` |

### Example: Create an About page

```json
{
  "shopId": "SHOP_ID",
  "slug": "about",
  "title": "About Us",
  "content": "<h2>Our Story</h2><p>Founded in 2020, we're passionate about bringing you the finest products...</p><p>Our mission is to make quality accessible to everyone.</p>",
  "status": "published",
  "showInHeader": true,
  "showInFooter": true,
  "displayOrder": 1,
  "seo": "{\"metaTitle\":\"About Us - SHOP_NAME\",\"metaDescription\":\"Learn about our story, mission, and values.\"}"
}
```

### Example: Create Terms & Conditions

```json
{
  "shopId": "SHOP_ID",
  "slug": "terms",
  "title": "Terms & Conditions",
  "content": "<h2>Terms & Conditions</h2><p>Last updated: April 2026</p><h3>1. General</h3><p>By using this website, you agree to these terms...</p><h3>2. Orders</h3><p>All orders are subject to availability...</p>",
  "status": "published",
  "showInHeader": false,
  "showInFooter": true,
  "displayOrder": 2
}
```

## Update Page

```graphql
mutation UpdatePage(
  $shopId: String!
  $pageId: String!
  $slug: String!
  $title: String!
  $content: String!
  $coverImage: String
  $status: String
  $showInHeader: Boolean
  $showInFooter: Boolean
  $displayOrder: Int
  $seo: String
) {
  updatePage(
    shopId: $shopId
    pageId: $pageId
    slug: $slug
    title: $title
    content: $content
    coverImage: $coverImage
    status: $status
    showInHeader: $showInHeader
    showInFooter: $showInFooter
    displayOrder: $displayOrder
    seo: $seo
  ) { id slug title status }
}
```

> **Important**: All fields are required in the mutation, but you should fetch the current page first and preserve unchanged fields.

### Workflow: Update Existing Page

1. **Fetch** — Get current page via `getPage(pageId: $pageId)`
2. **Modify** — Change only the fields the user requested, keep the rest
3. **Save** — Call `updatePage` with all fields (modified + preserved)
4. **Verify** — Re-fetch to confirm

## Delete Page

> ⚠️ **CONFIRMATION REQUIRED** — Show page title and slug before deleting. Warn that the URL will 404. Wait for explicit "Yes".

```graphql
mutation DeletePage($shopId: String!, $pageId: String!) {
  deletePage(shopId: $shopId, pageId: $pageId)
}
```

---

## Common Page Templates

AI can generate content for these standard pages:

| Page | Slug | Suggested Settings |
|---|---|---|
| About Us | `about` | `showInHeader: true`, `showInFooter: true` |
| Contact | `contact` | `showInHeader: true`, `showInFooter: true` |
| Terms & Conditions | `terms` | `showInFooter: true` |
| Privacy Policy | `privacy` | `showInFooter: true` |
| FAQ | `faq` | `showInHeader: true` |
| Shipping Info | `shipping` | `showInFooter: true` |
| Returns & Refunds | `returns` | `showInFooter: true` |
| Size Guide | `size-guide` | `showInHeader: false` |
| Our Story | `our-story` | `showInHeader: true` |

---

## Agent Notes

- **Slug rules**: Must be unique per shop. Use lowercase, hyphens for spaces, no special characters. E.g. `"terms-and-conditions"` not `"Terms & Conditions"`.
- **Content format**: HTML strings. Use semantic HTML (`<h2>`, `<h3>`, `<p>`, `<ul>`, `<li>`, `<strong>`, `<a>`) for well-structured pages.
- **SEO as JSON string**: The `seo` field is a JSON **string** (not a JSON object). Serialize it: `'{"metaTitle":"...","metaDescription":"..."}'`.
- **AI content generation**: When a user asks "write an about page" or "create terms", generate appropriate content for their shop's business type. Use the shop's name, category, and existing info to personalize.
- **Don't duplicate**: Check if a page with the same slug already exists before creating. If it exists, offer to update it instead.
