# Which components to support in PHP

Part of the exploration for [WordPress/gutenberg#84036](https://github.com/WordPress/gutenberg/issues/84036): making `@wordpress/ui` components available on PHP admin screens.

The goal: decide what parity means. Not every `@wordpress/ui` component needs a PHP version. This doc lists them all, then marks the ones the coat of paint restyled and the ones WooCommerce leans on most, to find a first set worth supporting.

## How to read the "Kind" column

-   **Static**: markup and CSS only, no JS. Works today with the build and `component.json` (like Button and Card).
-   **Toggle**: one on/off state. Works with hydration, and would also work with the small toggle script.
-   **Interactive**: needs real behavior (focus, keyboard, positioning, popups mounted in the browser). Only hydration covers it.
-   **Form**: a field. Native ones (`<input>`, `<textarea>`) work static. Base UI ones (Checkbox, Radio, Switch, Select) render custom markup with a hidden input, so they need hydration to be usable. All of them still post a real field with the `name` you give them.

## All `@wordpress/ui` components

| Component | Kind | Coat of paint | WooCommerce use |
| --- | --- | --- | --- |
| Button | Static | Yes (buttons) | Very high |
| ButtonLink | Static | Yes (buttons) | High (links styled as buttons) |
| IconButton | Static | Yes (buttons) | Medium |
| Link | Static | Typography | High |
| Text | Static | Yes (typography) | High |
| Stack | Static | Yes (spacing) | High (layout) |
| Card | Static | Yes (cards, metaboxes, dashboard widgets) | High (metaboxes) |
| Notice | Static (dismiss needs JS) | Yes (notices) | Very high |
| Badge | Static | No | Very high (order statuses) |
| EmptyState | Static | No | Medium (list screens with no items) |
| Spinner | Static | No | Medium |
| Skeleton | Static | No | Low |
| Icon | Static | No | Medium |
| VisuallyHidden | Static | No | Medium |
| Avatar | Static | No | Low |
| Breadcrumb | Static | No | Low |
| Meter | Static | No | Low |
| Progress | Static | No | Low (setup task lists) |
| KeyboardShortcutDisplay | Static | No | Low |
| CollapsibleCard | Toggle | Yes (collapsible metaboxes) | High (metaboxes open and close) |
| Collapsible | Toggle | No | Medium |
| Tooltip | Interactive | No | Very high (help tips, but see the infotip below) |
| Tabs | Interactive | No | High (settings tabs, product data) |
| Dialog | Interactive | No | High (modals) |
| AlertDialog | Interactive | No | Medium (confirm before deleting) |
| Popover | Interactive | No | Low |
| Menu | Interactive | No | Medium (row actions, "more" menus) |
| Drawer | Interactive | No | Low |
| Calendar | Interactive | No | Medium (date ranges in reports, sale dates) |
| InputControl | Form, native | Yes (inputs) | Very high |
| TextareaControl | Form, native | Yes (inputs) | Medium |
| SelectControl | Form, Base UI | Yes (inputs) | High |
| CheckboxControl | Form, Base UI | Yes (inputs) | High |
| RadioGroupControl | Form, Base UI | Yes (inputs) | Low |
| SwitchControl | Form, Base UI | No | Medium (enable/disable toggles) |
| SearchableSelectControl | Form, interactive | No | Very high (product, customer and page search) |
| SearchableChipSelectControl | Form, interactive | No | Medium (multi-select countries) |

## What the coat of paint restyled

The WordPress 7.0 admin reskin ([#64308](https://core.trac.wordpress.org/ticket/64308)) was visual only, CSS on top of the existing markup. Its scope:

-   Buttons and inputs ([#64547](https://core.trac.wordpress.org/ticket/64547))
-   Notices ([#64548](https://core.trac.wordpress.org/ticket/64548))
-   Cards, metaboxes and dashboard widgets ([#64549](https://core.trac.wordpress.org/ticket/64549))
-   Admin tables (a light refresh; there's no table in `@wordpress/ui`, DataViews covers it on the React side)
-   Typography and spacing

So the most used pieces of classic wp-admin map to Button, InputControl, Notice, Card, Text and Stack. All of them are static, except Notice's dismiss button.

## What WooCommerce uses most

From searching WooCommerce 11.2.1's classic admin PHP (`includes/admin`, `src/Internal/Admin`). It counts how often each pattern shows up in the code, not on screens, so it's a rough ranking rather than exact numbers:

| Pattern | Use | Maps to |
| --- | --- | --- |
| Form fields: raw `<input>`, `<select>` and `<textarea>`, `woocommerce_wp_*` helpers, settings fields | Highest | Form controls |
| Elements with a `button` class | Very high | Button, ButtonLink |
| `wc_help_tip()` help tips | Very high | Infotip |
| Status marks (`order-status`, `<mark>`) | High | Badge |
| Backbone modals | High | Dialog |
| Enhanced selects (product, customer, page search) | High | SearchableSelectControl |
| Notice markup | Medium | Notice |
| Tabs (`nav-tab`, product data tabs) | Medium | Tabs |
| Metaboxes | Medium | CollapsibleCard |
| Blank states | Medium | EmptyState |

Form fields are the most used pattern once you count every way WooCommerce prints one, not just its helpers. In the settings pages, checkbox, text and select are the most common types, followed by textarea, color, number and page select with search.

Metaboxes in WordPress are collapsible by default (core's `postbox` script adds the toggle), so every metabox is a CollapsibleCard candidate, not just a Card.

Extensions mostly build settings pages with the same Settings API and field helpers, add metaboxes to orders and products, and show notices. Their UI is a smaller copy of core WooCommerce's: fields, buttons, help tips, notices and metaboxes.

## Suggested first set

**1. Static (no JS, ready with what's there today):** Button, ButtonLink, IconButton, Link, Text, Stack, Card, Notice (without dismiss), Badge, EmptyState, Spinner, InputControl, TextareaControl.

This covers everything the coat of paint restyled, plus the WooCommerce patterns that need no JS (status badges, blank states). It's the cheapest set, and the highest use.

**2. Interactive, if hydration is accepted:** Infotip, CollapsibleCard, Tabs, Dialog, and the Base UI form controls (Checkbox, Select, Switch).

Help tips are the strongest case: they're all over WooCommerce, and they can't work without JS. Our Tooltip is a visual label only (hidden from screen readers, off on touch devices), so the design system's answer for help tips is an infotip: a Popover that opens on hover. It's a drop-in for `wc_help_tip()`, and the best component to test the interactive direction on.

**3. Later, on request:** Menu, AlertDialog, Calendar. SearchableSelectControl and SearchableChipSelectControl already work in the prototype with a native `<select>` fallback, but searching on the server (products, customers, pages) needs its own data contract.

**Leave React-only:** Drawer, Avatar, Breadcrumb, Meter, Progress, Skeleton, Popover on its own, Collapsible on its own.

## Open questions

-   Are Base UI checkboxes and switches acceptable on screens without JS? They post their starting value but can't be changed, and labels are only linked to them after JS loads. Radio and the searchable selects fall back to native fields.
-   Tables: the coat of paint restyled them, but `@wordpress/ui` has none. Is that left to `WP_List_Table` plus CSS?
-   Is the WooCommerce team's own list of needs close to this ranking?
