**12. Web API** (https://devdocs.magento.com/guides/v2.3/extension-dev-guide/service-contracts/service-to-web-service.html)

General workflow (for custom entity)
1. Create CRUD (Create, Read, Update, Delete). In Magento 2 CRUD includes:
  - Entity Model
  - Entity ResourceModel
  - Entity Resource Collection
  - Entity Data Interface (see "Service contracts"). Entity Model should Implement this Interface.
  - Entity Repository. For full web_api functionality, Repository shoud have following methods:
    - getList()
    - getById()
    - save() - for new entity
    - update() - for existing entity
    - delete()
  - Entity Repository Interface. Entity Repository should implement this Interface.

  Don't forget to configure preferences in `di.xml`.

2. Create etc/webapi.xml
```xml
<?xml version="1.0"?>
<routes xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:module:Magento_Webapi:etc/webapi.xsd">
    <route method="GET" url="/V1/study/wislist-item/">
        <service class="Study\WishlistWebapi\Api\WishlistItemRepositoryInterface" method="getList"/>
        <resources>
            <resource ref="self"/>
        </resources>
        <data>
            <parameter name="customerId" force="true">%customer_id%</parameter>
        </data>
    </route>
</routes>
```
  - `route` node contains api route configuration. `url` - endpoint, `method` - any HTTP `method`. In general case:
    - GET for `getList()` and `getById()` method
    - POST for `save()`
    - PUT for `update()`
    - DELETE for `delete()`
  - `service` - class and method of interface which processes api request.
  - `resource` node - can be `self`, `anonymous` or any admin ACL resource.

To get data using api with `self` resource, customer token must be passed as authorization header (https://devdocs.magento.com/guides/v2.3/get-started/authentication/gs-authentication-token.html). You can get customer token by making request with following params:
  - Method: POST
  - Url: `<your-website-url>/rest/V1/integration/customer/token`
  - Body params: {"username": "`<your customer email>`", "password": "`your customer password`"}

Response should contain token. Now you can add header "Authorization: Bearer `<Your customer token>`" to request with `self` resource and get access to data assigned to this customer. 

Admin token can be get the simular way. Endpoint with `anonymous` resource is accessable for everyone.
  
  - `data` node - contains additional parameters.
    - `parameter` - in this example states that request must contain `customerId` param. This param is defined from customer token.

After doing this, request to address `https://<your-website-url>/V1/study/wishlist-item` with customer token passed as request header should lead you to method `getList()` of your Entity Repository.


  - You also can add aliases for routes and replace default routes. Replacement must be defined in `etc/webapi_async.xml`:
```xml
<services xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:module:Magento_WebapiAsync:etc/webapi_async.xsd">
    <route url="V1/widgets" method="POST" alias="createWidget" />
</services>
```

All requests made on endpoint POST /createWidget will be forwarded to POST V1/widgets

**NOTICE:** Annotations in Repository Interface are VERY important, first of all the `@return` value. Without correct annotations Magento API Processor (`Magento\Framework\Webapi\ServiceInputProcessor`) will be unable to convert JSON array from your request body (in POST and PUT requests) into corresponding object. General rules for annotations in Repository Interface:
- `@return` and `@param` instruction must contain FULL CLASS NAME. Class aliases won't work.
- `getById()`, `save()` and `update()` methods return Entity.
- `deleteById()` method returns bool. 
- `getList()` method annotation should look like `Entity[]`. This helps Processor to understand that result is array, containing Entities:
    - `@return \Study\WishlistWebapi\Api\Data\WishlistItemInterface[]`
