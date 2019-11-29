# Clear directories during development
(https://devdocs.magento.com/guides/v2.3/howdoi/php/php_clear-dirs.html)

Action | Directories to clear
-------|---------------------
Change a class if there is a plugin related to it | generated/metadata, generated/code
A change that results on changing factories and proxies | generated/metadata, generated/code
Changes in `di.xml` | generated/metadata, generated/code
Add, remove, enable or disable module | generated/metadata, generated/code, var/cache, var/page_cache
Add or edit a layout or theme | var/view_preprocesses, var/cache, var/page_cache
Changes in LESS templates | var/view_preprocessed, var/cache, var/page_cache
Changes in `js` or `html` files | pub/static
Add or edit cms page, block | var/cache, var/page_cache
Change Magento config via admin panel | var/cache, var/page_cache

