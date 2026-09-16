# WordPress API response fields

AspireCloud selects WordPress response fields by action, then applies the caller's
`fields` overrides. This applies to `query_plugins` and `plugin_information` on
`/plugins/info/1.2`, and `query_themes` and `theme_information` on
`/themes/info/1.1` and `/themes/info/1.2`.

## Defaults and overrides

| Action | Default selection |
| --- | --- |
| `query_plugins` | Common plugin metadata plus `description`, `short_description`, `icons` and `downloaded`. No sections, release history, contributors, screenshots or other detail-only fields. |
| `plugin_information` | Common metadata and detail fields, including sections and versions. The four listing-only fields above are omitted unless requested. |
| `query_themes` | Identity, preview, author, screenshot, description, rating, number of ratings and homepage. |
| `theme_information` | Identity, preview, author, screenshot, sections, rating, number of ratings, downloads, download link, update dates, homepage and tags. |

Theme API 1.2 additionally enables `requires`, `requires_php`, `is_commercial`,
`is_community`, `external_repository_url`, `external_support_url` and extended
author objects. Theme details also enable `creation_time` and `reviews_url` in
1.2. API 1.1 defaults to the author's username; `extended_author` can override
either version's default.

Both nested `request[fields]` and top-level `fields` query parameters are accepted.
Supported forms are associative flags (`true`/`false`, `1`/`0`, including their
string representations), lists of enabled field names, and comma-separated field
names. Lists add to the action defaults; they do not act as exclusive whitelists.
Unknown fields and malformed flags are ignored. Explicit valid overrides win over
the defaults. The official input name `downloadlink` maps to `download_link` and
takes precedence when both names are supplied.

- `name`, `slug`, `version`, `ac_origin` and `ac_created` remain present.
- Plugin `reviews=false` removes only `sections.reviews`; `sections=false` omits
  the whole section map. Plugin `rating=false` does not disable `num_ratings`.
- Theme `sections=true` suppresses root `description`. Theme `rating=false` also
  disables `num_ratings`. `last_updated` selects both date representations, with
  `last_updated_time` available as an explicit independent override.
- Existing field value types, empty-value conventions and rewritten download URLs
  are preserved. Omitting `versions` does not prevent the download URL accessor
  from consulting stored versions when it needs a versioned URL.
- Fields not implemented by the existing response DTOs are not synthesized.

## WordPress callers

The core Add Plugins listing does not send a `fields` map, so action defaults
matter. The AJAX plugin and theme installers explicitly send `sections=false`;
these flags must be honored. Plugin dependency lookups request
`short_description` and `icons`. The Add Themes AJAX handler explicitly enables
fields including `downloadlink`, `downloaded`, `last_updated` and `tags`, while
disabling `sections`. The Customizer requests `reviews_url` on top of defaults.

These requests are covered by `tests/Feature/API/WpOrg/ResponseFieldsTest.php`.
The field normalizer and theme dependencies have additional unit coverage in
`tests/Unit/Values/ResponseFieldsTest.php`.

## FAIR and legacy compatibility

Field selection applies only to the WordPress response envelope. With
`feature.underscore_fair_hack` enabled and a truthy `_fair` query argument,
`InlineFairMetadata` still attaches the complete stored package document. A
`fields[_fair]=0` override cannot remove it. Releases, signatures and extension
properties inside that document are not filtered or rewritten by field selection.
Without both switches, no inline document is added.

FAIR Connect adds `_fair` when redirecting requests to its default repository and
uses that document for the FAIR plugin information screen. `/packages` responses
and raw metadata storage are unchanged. Opting into inline FAIR metadata can
therefore still produce a large response; trimming the signed document is not
part of WordPress field selection.

The plugin 1.0 JSON route, theme 1.0 serialized responses, update endpoints,
search selection and pagination retain their existing behavior. No WordPress or
FAIR Connect modification is required for this server-side field selection.

## Reference implementations

- [WordPress AJAX handlers](https://github.com/WordPress/wordpress-develop/blob/7.1/src/wp-admin/includes/ajax-actions.php)
- [WordPress plugin list table](https://github.com/WordPress/wordpress-develop/blob/7.1/src/wp-admin/includes/class-wp-plugin-install-list-table.php)
- [WordPress theme API server](https://github.com/WordPress/wordpress.org/blob/cd7e37164d42543ace7c206d6c76f5ebce50a654/wordpress.org/public_html/wp-content/plugins/theme-directory/class-themes-api.php)
- [FAIR Connect repository redirection](https://github.com/fairpm/fair-plugin/blob/aae5c4d6e22ee7d2ce80a60e272ae3b2fc08c179/inc/default-repo/namespace.php)
- [FAIR Connect plugin information handling](https://github.com/fairpm/fair-plugin/blob/aae5c4d6e22ee7d2ce80a60e272ae3b2fc08c179/inc/packages/admin/namespace.php)
