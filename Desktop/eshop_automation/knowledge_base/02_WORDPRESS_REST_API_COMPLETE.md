# WordPress REST API - Complete Reference

## Overview

Base URL: `https://yoursite.com/wp-json/wp/v2/`

Authentication: Application Passwords, JWT, OAuth, Cookie/Nonce

---

## 1. POSTS API

### Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/posts` | List posts |
| GET | `/posts/{id}` | Get post |
| POST | `/posts` | Create post |
| PUT/PATCH | `/posts/{id}` | Update post |
| DELETE | `/posts/{id}` | Delete post |

### Post Properties

```json
{
  "id": 1,
  "date": "2024-01-15T10:00:00",
  "date_gmt": "2024-01-15T08:00:00",
  "guid": {"rendered": "https://site.com/?p=1"},
  "modified": "2024-01-20T15:00:00",
  "modified_gmt": "2024-01-20T13:00:00",
  "slug": "post-slug",
  "status": "publish|future|draft|pending|private",
  "type": "post",
  "link": "https://site.com/post-slug/",
  "title": {"rendered": "Post Title"},
  "content": {"rendered": "<p>Content HTML</p>", "protected": false},
  "excerpt": {"rendered": "<p>Excerpt</p>", "protected": false},
  "author": 1,
  "featured_media": 100,
  "comment_status": "open|closed",
  "ping_status": "open|closed",
  "sticky": false,
  "template": "",
  "format": "standard|aside|chat|gallery|link|image|quote|status|video|audio",
  "meta": [],
  "categories": [1, 2],
  "tags": [5, 6]
}
```

### Query Parameters for GET /posts

| Parameter | Type | Description |
|-----------|------|-------------|
| `context` | string | view, edit, embed |
| `page` | integer | Current page |
| `per_page` | integer | Items per page (max 100) |
| `search` | string | Search term |
| `after` | string | After date (ISO8601) |
| `before` | string | Before date (ISO8601) |
| `author` | array | Filter by author IDs |
| `author_exclude` | array | Exclude authors |
| `exclude` | array | Exclude post IDs |
| `include` | array | Include post IDs |
| `offset` | integer | Offset |
| `order` | string | asc, desc |
| `orderby` | string | author, date, id, include, modified, parent, relevance, slug, title |
| `slug` | array | Filter by slug |
| `status` | array | Filter by status |
| `categories` | array | Filter by category IDs |
| `categories_exclude` | array | Exclude categories |
| `tags` | array | Filter by tag IDs |
| `tags_exclude` | array | Exclude tags |
| `sticky` | boolean | Filter sticky posts |

---

## 2. PAGES API

### Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/pages` | List pages |
| GET | `/pages/{id}` | Get page |
| POST | `/pages` | Create page |
| PUT/PATCH | `/pages/{id}` | Update page |
| DELETE | `/pages/{id}` | Delete page |

### Page Properties

```json
{
  "id": 10,
  "date": "2024-01-15T10:00:00",
  "slug": "page-slug",
  "status": "publish",
  "type": "page",
  "link": "https://site.com/page-slug/",
  "title": {"rendered": "Page Title"},
  "content": {"rendered": "<p>Content</p>"},
  "excerpt": {"rendered": ""},
  "author": 1,
  "featured_media": 0,
  "parent": 0,
  "menu_order": 0,
  "comment_status": "closed",
  "ping_status": "closed",
  "template": "template-custom.php",
  "meta": []
}
```

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `parent` | array | Filter by parent page |
| `parent_exclude` | array | Exclude by parent |
| `menu_order` | integer | Filter by menu order |

---

## 3. MEDIA API

### Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/media` | List media |
| GET | `/media/{id}` | Get media item |
| POST | `/media` | Upload media |
| PUT/PATCH | `/media/{id}` | Update media |
| DELETE | `/media/{id}` | Delete media |

### Media Properties

```json
{
  "id": 100,
  "date": "2024-01-15T10:00:00",
  "slug": "image-name",
  "status": "inherit",
  "type": "attachment",
  "link": "https://site.com/image-name/",
  "title": {"rendered": "Image Name"},
  "author": 1,
  "description": {"rendered": "Description"},
  "caption": {"rendered": "Caption"},
  "alt_text": "Alt text",
  "media_type": "image|file",
  "mime_type": "image/jpeg",
  "media_details": {
    "width": 1920,
    "height": 1080,
    "file": "2024/01/image.jpg",
    "filesize": 150000,
    "sizes": {
      "thumbnail": {
        "file": "image-150x150.jpg",
        "width": 150,
        "height": 150,
        "mime_type": "image/jpeg",
        "source_url": "https://site.com/wp-content/uploads/2024/01/image-150x150.jpg"
      },
      "medium": {...},
      "large": {...},
      "full": {...}
    }
  },
  "post": 1,
  "source_url": "https://site.com/wp-content/uploads/2024/01/image.jpg"
}
```

### Upload Media (POST)

```
POST /wp-json/wp/v2/media
Content-Type: image/jpeg
Content-Disposition: attachment; filename="image.jpg"

[binary data]
```

---

## 4. USERS API

### Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/users` | List users |
| GET | `/users/{id}` | Get user |
| GET | `/users/me` | Get current user |
| POST | `/users` | Create user |
| PUT/PATCH | `/users/{id}` | Update user |
| DELETE | `/users/{id}` | Delete user |

### User Properties

```json
{
  "id": 1,
  "username": "admin",
  "name": "Admin User",
  "first_name": "Admin",
  "last_name": "User",
  "email": "admin@site.com",
  "url": "https://site.com",
  "description": "Bio here",
  "link": "https://site.com/author/admin/",
  "locale": "en_US",
  "nickname": "admin",
  "slug": "admin",
  "registered_date": "2020-01-01T00:00:00",
  "roles": ["administrator"],
  "capabilities": {...},
  "extra_capabilities": {...},
  "avatar_urls": {
    "24": "https://gravatar.com/...",
    "48": "https://gravatar.com/...",
    "96": "https://gravatar.com/..."
  },
  "meta": []
}
```

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `roles` | array | Filter by roles |
| `who` | string | Filter by authors |
| `has_published_posts` | boolean | Users with posts |

---

## 5. CATEGORIES API

### Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/categories` | List categories |
| GET | `/categories/{id}` | Get category |
| POST | `/categories` | Create category |
| PUT/PATCH | `/categories/{id}` | Update category |
| DELETE | `/categories/{id}` | Delete category |

### Category Properties

```json
{
  "id": 1,
  "count": 25,
  "description": "Category description",
  "link": "https://site.com/category/slug/",
  "name": "Category Name",
  "slug": "category-slug",
  "taxonomy": "category",
  "parent": 0,
  "meta": []
}
```

---

## 6. TAGS API

### Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/tags` | List tags |
| GET | `/tags/{id}` | Get tag |
| POST | `/tags` | Create tag |
| PUT/PATCH | `/tags/{id}` | Update tag |
| DELETE | `/tags/{id}` | Delete tag |

### Tag Properties

```json
{
  "id": 5,
  "count": 10,
  "description": "Tag description",
  "link": "https://site.com/tag/slug/",
  "name": "Tag Name",
  "slug": "tag-slug",
  "taxonomy": "post_tag",
  "meta": []
}
```

---

## 7. COMMENTS API

### Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/comments` | List comments |
| GET | `/comments/{id}` | Get comment |
| POST | `/comments` | Create comment |
| PUT/PATCH | `/comments/{id}` | Update comment |
| DELETE | `/comments/{id}` | Delete comment |

### Comment Properties

```json
{
  "id": 1,
  "post": 100,
  "parent": 0,
  "author": 0,
  "author_name": "John Doe",
  "author_email": "john@example.com",
  "author_url": "https://johndoe.com",
  "author_ip": "192.168.1.1",
  "author_user_agent": "Mozilla/5.0...",
  "date": "2024-01-15T10:00:00",
  "content": {"rendered": "<p>Comment content</p>"},
  "link": "https://site.com/post/#comment-1",
  "status": "approved|hold|spam|trash",
  "type": "comment",
  "author_avatar_urls": {...},
  "meta": []
}
```

---

## 8. TAXONOMIES API

### Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/taxonomies` | List taxonomies |
| GET | `/taxonomies/{slug}` | Get taxonomy |

### Taxonomy Properties

```json
{
  "name": "Categories",
  "slug": "category",
  "description": "",
  "types": ["post"],
  "hierarchical": true,
  "rest_base": "categories",
  "rest_namespace": "wp/v2",
  "visibility": {
    "public": true,
    "publicly_queryable": true,
    "show_ui": true,
    "show_admin_column": true,
    "show_in_nav_menus": true,
    "show_in_quick_edit": true
  }
}
```

---

## 9. POST TYPES API

### Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/types` | List post types |
| GET | `/types/{slug}` | Get post type |

### Post Type Properties

```json
{
  "description": "",
  "hierarchical": false,
  "has_archive": true,
  "name": "Posts",
  "slug": "post",
  "taxonomies": ["category", "post_tag"],
  "rest_base": "posts",
  "rest_namespace": "wp/v2",
  "visibility": {
    "show_in_nav_menus": true,
    "show_ui": true
  }
}
```

---

## 10. CUSTOM POST TYPES

### WooCommerce Products via WP API

```
GET /wp-json/wp/v2/product
GET /wp-json/wp/v2/product/{id}
```

### Any Custom Post Type

```
GET /wp-json/wp/v2/{post_type}
```

Requires `show_in_rest` => true in register_post_type()

---

## 11. CUSTOM TAXONOMIES

### WooCommerce Product Categories

```
GET /wp-json/wp/v2/product_cat
```

### Any Custom Taxonomy

```
GET /wp-json/wp/v2/{taxonomy}
```

---

## 12. SETTINGS API

### Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/settings` | Get all settings |
| PUT/PATCH | `/settings` | Update settings |

### Available Settings

```json
{
  "title": "Site Title",
  "description": "Site Tagline",
  "url": "https://site.com",
  "email": "admin@site.com",
  "timezone": "Europe/Athens",
  "date_format": "F j, Y",
  "time_format": "g:i a",
  "start_of_week": 1,
  "language": "el",
  "use_smilies": true,
  "default_category": 1,
  "default_post_format": "",
  "posts_per_page": 10,
  "default_ping_status": "open",
  "default_comment_status": "open"
}
```

---

## 13. BLOCK TYPES API

### Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/block-types` | List block types |
| GET | `/block-types/{namespace}/{name}` | Get block type |

---

## 14. BLOCK RENDERER API

### Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/block-renderer/{name}` | Render block |
| POST | `/block-renderer/{name}` | Render block with attributes |

---

## 15. SEARCH API

### Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/search` | Search content |

### Query Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `search` | string | Search term |
| `type` | string | post, term, post-format |
| `subtype` | string | post, page, category, tag |
| `per_page` | integer | Results per page |

---

## 16. PLUGINS API (WP 5.5+)

### Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/plugins` | List plugins |
| GET | `/plugins/{plugin}` | Get plugin |
| POST | `/plugins` | Install plugin |
| PUT | `/plugins/{plugin}` | Activate/deactivate |
| DELETE | `/plugins/{plugin}` | Delete plugin |

### Plugin Properties

```json
{
  "plugin": "plugin-folder/plugin-file.php",
  "status": "active|inactive",
  "name": "Plugin Name",
  "plugin_uri": "https://plugin-site.com",
  "author": "Author Name",
  "author_uri": "https://author-site.com",
  "description": {
    "raw": "Description",
    "rendered": "<p>Description</p>"
  },
  "version": "1.0.0",
  "network_only": false,
  "requires_wp": "5.0",
  "requires_php": "7.4",
  "text_domain": "plugin-textdomain"
}
```

---

## 17. APPLICATION PASSWORDS API

### Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/users/{user_id}/application-passwords` | List passwords |
| POST | `/users/{user_id}/application-passwords` | Create password |
| DELETE | `/users/{user_id}/application-passwords/{uuid}` | Delete password |
| DELETE | `/users/{user_id}/application-passwords` | Delete all passwords |

### Create Application Password

```json
POST /wp-json/wp/v2/users/1/application-passwords
{
  "name": "n8n Integration"
}

Response:
{
  "uuid": "xxxxx",
  "app_id": "",
  "name": "n8n Integration",
  "password": "xxxx xxxx xxxx xxxx xxxx xxxx",
  "created": "2024-01-15T10:00:00",
  "last_used": null,
  "last_ip": null
}
```

---

## 18. MENUS API (WP 5.9+)

### Navigation Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/navigation` | List navigation menus |
| GET | `/navigation/{id}` | Get navigation |
| POST | `/navigation` | Create navigation |
| PUT | `/navigation/{id}` | Update navigation |
| DELETE | `/navigation/{id}` | Delete navigation |

### Menu Items

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/menu-items` | List menu items |
| GET | `/menu-items/{id}` | Get menu item |
| POST | `/menu-items` | Create menu item |
| PUT | `/menu-items/{id}` | Update menu item |
| DELETE | `/menu-items/{id}` | Delete menu item |

### Menu Locations

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/menu-locations` | List locations |
| GET | `/menu-locations/{location}` | Get location |

---

## 19. GLOBAL STYLES API

### Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/global-styles/themes/{stylesheet}` | Get theme styles |
| PUT | `/global-styles/themes/{stylesheet}` | Update theme styles |
| GET | `/global-styles/{id}` | Get global styles |
| PUT | `/global-styles/{id}` | Update global styles |

---

## 20. PATTERN DIRECTORY API

### Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/pattern-directory/patterns` | Get patterns |

---

## 21. ADVANCED CUSTOM FIELDS (ACF) INTEGRATION

### ACF Fields in REST API

Enable ACF fields in REST API:

1. Field Group Settings → Show in REST API = Yes
2. Or via filter:
```php
add_filter('acf/rest_api/field_settings/show_in_rest', '__return_true');
```

### Reading ACF Data

```
GET /wp-json/wp/v2/posts/{id}

Response includes:
{
  ...
  "acf": {
    "custom_field_1": "value",
    "custom_field_2": "value",
    "repeater_field": [
      {"sub_field_1": "value"},
      {"sub_field_2": "value"}
    ],
    "image_field": {
      "ID": 100,
      "url": "https://...",
      "sizes": {...}
    }
  }
}
```

### Updating ACF Data

```json
PUT /wp-json/wp/v2/posts/{id}
{
  "acf": {
    "custom_field_1": "new value",
    "repeater_field": [
      {"sub_field_1": "new value"}
    ]
  }
}
```

### ACF Options Page

```
GET /wp-json/acf/v3/options/{option_page}
PUT /wp-json/acf/v3/options/{option_page}
```

---

## 22. YOAST SEO INTEGRATION

### Yoast Meta in REST API

```
GET /wp-json/wp/v2/posts/{id}

Response includes:
{
  ...
  "yoast_head": "<title>...</title><meta name='description'...",
  "yoast_head_json": {
    "title": "SEO Title",
    "description": "Meta description",
    "robots": {
      "index": "index",
      "follow": "follow"
    },
    "canonical": "https://site.com/post/",
    "og_locale": "en_US",
    "og_type": "article",
    "og_title": "OG Title",
    "og_description": "OG Description",
    "og_url": "https://site.com/post/",
    "og_site_name": "Site Name",
    "og_image": [...],
    "twitter_card": "summary_large_image",
    "schema": {...}
  }
}
```

### Update Yoast Meta

```json
PUT /wp-json/wp/v2/posts/{id}
{
  "meta": {
    "_yoast_wpseo_title": "Custom SEO Title",
    "_yoast_wpseo_metadesc": "Custom meta description",
    "_yoast_wpseo_focuskw": "focus keyword",
    "_yoast_wpseo_canonical": "https://site.com/canonical-url/"
  }
}
```

---

## 23. AUTHENTICATION METHODS

### 1. Application Passwords (Recommended)

```
Authorization: Basic base64(username:application_password)
```

### 2. JWT Authentication (with plugin)

```
POST /wp-json/jwt-auth/v1/token
{
  "username": "admin",
  "password": "password"
}

Response:
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user_email": "admin@site.com",
  "user_nicename": "admin",
  "user_display_name": "Admin"
}

Then use:
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

### 3. Cookie Authentication (for plugins/themes)

```javascript
// In WordPress admin
fetch('/wp-json/wp/v2/posts', {
  headers: {
    'X-WP-Nonce': wpApiSettings.nonce
  }
});
```

### 4. OAuth 1.0a (with plugin)

```
OAuth oauth_consumer_key="xxx",
      oauth_token="xxx",
      oauth_signature_method="HMAC-SHA1",
      oauth_signature="xxx",
      oauth_timestamp="xxx",
      oauth_nonce="xxx"
```

---

## 24. PAGINATION & HEADERS

### Response Headers

```
X-WP-Total: 100
X-WP-TotalPages: 10
Link: <https://site.com/wp-json/wp/v2/posts?page=2>; rel="next",
      <https://site.com/wp-json/wp/v2/posts?page=10>; rel="last"
```

### Pagination Parameters

| Parameter | Default | Max | Description |
|-----------|---------|-----|-------------|
| `page` | 1 | - | Page number |
| `per_page` | 10 | 100 | Items per page |
| `offset` | 0 | - | Skip items |

---

## 25. ERROR RESPONSES

### Standard Error Format

```json
{
  "code": "rest_post_invalid_id",
  "message": "Invalid post ID.",
  "data": {
    "status": 404
  }
}
```

### Common Error Codes

| Code | HTTP | Description |
|------|------|-------------|
| `rest_forbidden` | 403 | No permission |
| `rest_forbidden_context` | 403 | Cannot use context |
| `rest_post_invalid_id` | 404 | Invalid post ID |
| `rest_user_invalid_id` | 404 | Invalid user ID |
| `rest_term_invalid` | 404 | Invalid term |
| `rest_no_route` | 404 | Route not found |
| `rest_cannot_create` | 403 | Cannot create |
| `rest_cannot_edit` | 403 | Cannot edit |
| `rest_cannot_delete` | 403 | Cannot delete |
| `rest_invalid_param` | 400 | Invalid parameter |

---

## 26. EMBEDDING & _FIELDS

### Embed Related Resources

```
GET /wp-json/wp/v2/posts?_embed

Response includes:
{
  ...
  "_embedded": {
    "author": [{user object}],
    "wp:featuredmedia": [{media object}],
    "wp:term": [
      [{category objects}],
      [{tag objects}]
    ]
  }
}
```

### Select Specific Fields

```
GET /wp-json/wp/v2/posts?_fields=id,title,link,categories
```

---

## 27. BATCH REQUESTS (Plugin Required)

Με το WP REST API - Batch plugin:

```json
POST /wp-json/wp/v2/batch/v1

{
  "requests": [
    {"path": "/wp/v2/posts/1", "method": "GET"},
    {"path": "/wp/v2/posts/2", "method": "GET"},
    {"path": "/wp/v2/posts", "method": "POST", "body": {"title": "New Post"}}
  ]
}
```

---

## 28. CACHING HEADERS

### Response Headers

```
Cache-Control: max-age=300
ETag: "xxxxx"
Last-Modified: Mon, 15 Jan 2024 10:00:00 GMT
```

### Conditional Requests

```
GET /wp-json/wp/v2/posts/1
If-None-Match: "xxxxx"
If-Modified-Since: Mon, 15 Jan 2024 10:00:00 GMT

Response: 304 Not Modified (if unchanged)
```

---

## 29. MULTISITE API

### Sites Endpoints (Super Admin)

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/sites` | List sites |
| GET | `/sites/{id}` | Get site |
| POST | `/sites` | Create site |
| PUT | `/sites/{id}` | Update site |
| DELETE | `/sites/{id}` | Delete site |

---

## 30. USEFUL DISCOVERY ENDPOINTS

### Root Endpoint

```
GET /wp-json/

Returns all available namespaces and routes
```

### Namespace Discovery

```
GET /wp-json/wp/v2/
GET /wp-json/wc/v3/
GET /wp-json/acf/v3/
```
