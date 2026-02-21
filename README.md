# WCS Price Sync

A WordPress plugin that keeps WooCommerce Subscription prices in step with their parent products — automatically when a product is saved, and manually via a bulk action on the subscription list. A configurable buffer window prevents price changes from landing too close to an upcoming renewal.

---

## Requirements

| Dependency | Version |
|---|---|
| WordPress | 6.0+ |
| WooCommerce | 8.0+ |
| WooCommerce Subscriptions | 5.0+ (premium) |
| PHP | 8.0+ |

---

## Installation

1. Copy `wcs-price-sync.php` into `wp-content/plugins/wcs-price-sync/`.
2. Activate the plugin from **Plugins → Installed Plugins**.
3. No settings page is required — the plugin reads the renewal notification offset already configured in **WooCommerce → Settings → Subscriptions** and uses it as the buffer window.

---

## How It Works

### Automatic sync on product save

When a subscription product is updated in the admin, the plugin hooks into `woocommerce_before_product_object_save` and propagates the new price to every active subscription that contains that product — provided those subscriptions are outside the buffer window (see below).

WooCommerce fires this hook twice per admin save (once for an autosave, once for the real save). The plugin uses a 30-second transient lock so it only acts on the second call, when the definitive price is available.

### The buffer window

The plugin reads the **renewal reminder offset** from WCS — the same "X days/weeks before renewal" setting that controls when reminder emails are sent. Any subscription whose next payment falls within that window is skipped during a sync. This ensures customers never receive a reminder quoting one price and are then charged a different one.

### Manual bulk sync

A **Sync Prices (with Buffer)** option is added to the bulk-action dropdown on **WooCommerce → Subscriptions**. Selecting one or more subscriptions and applying this action updates each line item to the current product price, again respecting the buffer window. A dismissible admin notice reports how many subscriptions were updated.

---

## Filters & Hooks

This plugin does not currently expose additional filters, but the following WordPress/WooCommerce hooks are used and can be unhooked if you need to modify behaviour in another plugin:

| Hook | Type | Priority | Description |
|---|---|---|---|
| `woocommerce_before_product_object_save` | action | 20 | Triggers automatic price sync |
| `admin_footer-edit.php` | action | default | Injects the bulk-action `<option>` |
| `admin_init` | action | 1 | Intercepts and processes the bulk action |
| `admin_notices` | action | default | Displays the post-sync count notice |

---

## Buffer Window Reference

The buffer duration is derived from the WCS setting at:  
**WooCommerce → Settings → Subscriptions → Customer Notifications → Renewal reminder**

| WCS setting | Buffer applied |
|---|---|
| 3 days | 259,200 seconds |
| 1 week | 604,800 seconds |
| 1 month | 2,592,000 seconds |

If the WCS classes needed to read this setting are unavailable, the buffer defaults to **zero** (no buffer) rather than failing silently with an incorrect value.

---

## Frequently Asked Questions

**Will this update prices on paused or cancelled subscriptions?**  
No. Both the automatic sync and the bulk action only act on subscriptions with an `active` status.

**What happens if a subscription has no next payment date?**  
It is skipped. Subscriptions with no upcoming payment (e.g. lifetime or already-expired) are never touched.

**Does this affect automatic (gateway-charged) renewals differently to manual ones?**  
No. The price sync logic is the same for both. Only the companion plugin [WCS Extended Renewal Reminders](../wcs-extended-renewal-reminders/) distinguishes between manual and automatic subscriptions.

**Can I set a different buffer period just for price syncing?**  
Not currently — the buffer is always read from the WCS notification offset setting. If you need an independent value, filter `wcs_get_notification_buffer_seconds()` return value from another plugin.

---

## Changelog

### 3.0
- Removed dual-notification scheduler (moved to **WCS Extended Renewal Reminders**).
- Renamed plugin to **WCS Price Sync** to reflect its focused scope.
- Minor code cleanup: consistent spacing, removed stray `error_log()` calls.

### 2.2
- Introduced buffer window derived from WCS notification settings.
- Added bulk-action handler with admin notice.

### 2.1
- Initial dual-save transient lock to handle WooCommerce's double-fire on product save.

---

## License

GPL v2 or later — see [https://www.gnu.org/licenses/gpl-2.0.html](https://www.gnu.org/licenses/gpl-2.0.html)

**Author:** Frey Mansikkaniemi · [frey.hk](https://frey.hk)
