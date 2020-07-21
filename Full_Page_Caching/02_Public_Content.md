**2. Public content**
(https://devdocs.magento.com/guides/v2.2/extension-dev-guide/cache/page-caching/public-content.html)

Cache behavior can be changed is controller:
```php
public function execute()
{
    $page = $this->pageFactory->create();
    //We are using HTTP headers to control various page caches (varnish, fastly, built-in php cache)
    $page->setHeader('Cache-Control', 'no-store, no-cache, must-revalidate, max-age=0', true);

    return $page;
}
```

Usually URL is used as cache key. However, Magento 2 urls are not unique enough to serve as cache keys. To make cache key totally unique *HTTP Context varibles are used*. 
This variables allow Magento to display *different content for same urls* based on:
- Customer group
- Selected language
- Selected store
- Selected currency
- Whether a customer is logged in or not

Method `Magento\Framework\App\Http\Context::getVaryString()` allows to to retrieve unique identifier for selected context.
