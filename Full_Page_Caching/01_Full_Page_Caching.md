**1. Full Page Caching (FPC)**
(https://devdocs.magento.com/guides/v2.2/extension-dev-guide/cache/page-caching.html)

All pages are cacheable by default. Exceptions are, for example, checkout, compare products page, cart and some others.
If any block on the page has attribute `cacheable="false"` the whole page is excluded from FPC.

It is possible to define two types of content by cache behavior:
- Public content is stored on *server* side and includes content which is available for *all* customers. Examples of public content are: header, footer, list of categories.
- Private content is stored on *client* side and is specific for each *individual* customer. Examples: wishlist, shopping cart, customer name.

Only `GET` and `HEAD` methods are cacheable.