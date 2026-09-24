# Bloomreach Discovery Pixel GTM Web Tag

**A sandboxed Google Tag Manager template for the Bloomreach Discovery pixel: page views, conversions, single page app views and event pixels, without Custom HTML.**

[![Created by Freek Kampen](https://img.shields.io/badge/Created%20by-Freek%20Kampen-455CE9)](https://freekkampen.com) [![Maintained by New North Digital](https://img.shields.io/badge/Maintained%20by-New%20North%20Digital-455CE9)](https://newnorth.nl/?utm_source=github&utm_medium=gtm-template&utm_campaign=bloomreach-web-tag)

## Features

- Loads Bloomreach's own tracker (`br-trk-{account}.js`) and sets `br_data`, exactly like the official snippet.
- One tag for all core pixels: page view (homepage, product, category, search, content, conversion, thematic, other), virtual page view for single page apps, add to cart, search submit, suggest click and quick view.
- Reads GA4 `ecommerce` from the dataLayer as a fallback: product ID on product pages and add to cart, and order ID, value, currency and basket on the conversion page.
- Handles GA4 catalogs where `item_id` is the SKU and `item_group_id` the product ID.
- Leaves empty fields out, so Bloomreach never receives `"undefined"` or `"null"` values.
- Test data and Integration mode (`debug=true`) switches for staging and validation. In GTM Preview, `debug=true` is added automatically.
- Page types are trimmed and lowercased (`Product ` becomes `product`). Values Bloomreach does not accept (for example `pdp` or an empty variable) are sent as `other`; with debug logging on, the console says so.
- Built-in Consent Mode gate on `analytics_storage`.

## Web or server?

There is also a [server-side version](https://github.com/newnorthdigital/bloomreach-server-tag). Use one or the other per site, never both: Bloomreach warns that sending the same events client-side and server-side for more than a few hours corrupts analytics and search performance.

## Installation

### From the Community Template Gallery
1. In a GTM web container, open **Templates → Tag Templates → Search Gallery**.
2. Search for **Bloomreach Discovery Pixel by New North** and add it.

### Manual installation
1. Download `template.tpl` from this repo.
2. In GTM: **Templates → New → ⋮ → Import**, select the file, and save.

## Setup guide

1. **Page view tag.** Pixel type **Page view**, your **Account ID**, and **Page type** set to a variable that returns the Bloomreach page type for the current page (a lookup table on page path or a dataLayer key works well). Fill the fields for each page type with variables. Fire it once per page load.
   - **Product and conversion pages:** the GA4 fallbacks read the dataLayer at the moment the tag fires. On the order confirmation page, exclude the page from the normal page view trigger and fire a Page view tag with page type `conversion` on the `purchase` event, after the dataLayer push. Otherwise the pixel goes out without order ID and basket. Same for product pages if you rely on the `view_item` items instead of variables.
   - If the basket has items but no value, the tag sends the sum of price × quantity, because Bloomreach's tracker drops a conversion without `basket_value`.
2. **Single page apps.** Add a second tag with Pixel type **Virtual page view** and fire it on route changes (History Change or a custom dataLayer event). If the tracker has not started yet, the virtual page view is sent as a normal page view. In a single page app, `ecommerce.items` can still hold the previous product, so fill product fields with variables instead of the GA4 fallback.
3. **Event tags.** One tag per event: **Add to cart** on your add_to_cart event, **Search submit** when a search is submitted, **Suggest click** when an autosuggest term is clicked, **Quick view** when a quick view opens. Event tags use the tracker loaded by the Page view tag. Bloomreach's tracker starts at the window load event (or right away if the page has already finished parsing), so tag sequencing is not enough: an event before that fails (the tag reports failure and logs why in preview). Clicks and interactions after the page has loaded are fine.
4. **Validate.** Open GTM Preview: the tag then sends debug events on its own, which show up within seconds in Event diagnostics, Integration mode. Bloomreach's Pixel Validator Chrome extension works too. Tick **Send as debug events** only to test outside Preview, and untick it before publishing. Tick **Mark as test data** on staging only.

## Field reference

| Field | Bloomreach parameter | Notes |
|---|---|---|
| Account ID | `acct_id` | Required. |
| Domain key | `domain_key` | Only for accounts with more than one product catalog. |
| View ID | `view_id` | Only for multi-view accounts. |
| User ID | `user_id` | Anonymised customer ID. Leave empty for guests. |
| Page type | `ptype` | homepage, product, category, search, content, conversion, thematic or other. Accepts a variable. |
| Page title | `title` | Defaults to `document.title`. |
| Product ID / name / SKU | `prod_id`, `prod_name`, `sku` | Product pages. Empty: first item in `ecommerce.items`. |
| Category ID / name | `cat_id`, `cat` | Category pages. |
| Search term | `search_term` | Search result pages. |
| Content item ID / name | `item_id`, `item_name` | Content pages. |
| Order ID, basket value, currency | `order_id`, `basket_value`, `currency` | Conversion page. Empty: `ecommerce.transaction_id`, `value`, `currency`. |
| Basket items | `basket` | Conversion page. Array in GA4 or Bloomreach shape. Empty: `ecommerce.items`. `is_conversion=1` is set automatically. |
| Search query / typed query | `q`, `aq` | Search submit and suggest click. |
| Catalogs | `catalogs` | Comma-separated names, or a variable returning Bloomreach's catalog array. Content pages, and search when you have a content catalog. |
| GA4 item ID mapping | | Whether GA4 `item_id` is the product ID or the SKU. |
| Additional parameters | any | E.g. `customer_tier`, `customer_country`, `customer_geo`, `customer_profile`, `tms`. |
| Send as debug events | `debug=true` | Integration mode in Event diagnostics. Automatic in GTM Preview. |
| Mark as test data | `test_data=true` | Staging only. |

## Permissions

- Injects scripts from `https://cdn.brcdn.com/v1/*`.
- Reads and writes the global `br_data`; reads and calls `BrTrk.getTracker`.
- Reads `ecommerce.*` from the dataLayer.
- Reads the page URL (for `orig_ref_url` on virtual page views).
- Reads `analytics_storage` consent state.
- Reads container data (to detect GTM Preview).
- Logs to the console in debug and preview mode only.

## Resources

- [Bloomreach pixel checklist](https://documentation.bloomreach.com/discovery/docs/pixel-checklist)
- [Pixel parameter reference](https://documentation.bloomreach.com/discovery/docs/pixel-reference)
- [Single page application tracking](https://documentation.bloomreach.com/discovery/docs/virtual-page-view-pixel)

## Author

Created and maintained by [Freek Kampen](https://freekkampen.com) at [New North Digital](https://newnorth.nl/?utm_source=github&utm_medium=gtm-template&utm_campaign=bloomreach-web-tag).

## License

Apache 2.0, see [LICENSE](LICENSE).
