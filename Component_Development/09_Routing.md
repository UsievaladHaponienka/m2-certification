 **9. Routing** (https://devdocs.magento.com/guides/v2.3/extension-dev-guide/routing.html)

In general, routing is the act of providing data from url request to corresponding class for processing.

Magento has multiple router and provides possibility to add custom router.

Main Magento router is Base router. This router resolves Action class by parsing request using standard magento routing pattern - `<website-url>/<module>/<controller>/<action>`. The idea of custom routers is to **set proper module, controller and action name if route pattern is different from standard.**

To create custom router you need:
- Register custom router in `di.xml`
```xml
<type name="Magento\Framework\App\RouterList">
    <arguments>
        <argument name="routerList" xsi:type="array">
            <item name="%name%" xsi:type="array">
                <item name="class" xsi:type="string">%classpath%</item>
                <item name="disable" xsi:type="boolean">false</item>
                <item name="sortOrder" xsi:type="string">%sortorder%</item>
            </item>
        </argument>
    </arguments>
</type>
``` 
- Create router class which implements \Magento\Framework\App\RouterInterface. This class must implement method `match()`.
- Create route in `routes.xml` which uses custom router.
```xml
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:App/etc/routes.xsd">
    <router id="%routerId%">
        <route id="%routeId%" frontName="%frontName%">
            <module name="%moduleName%"/>
        </route>
    </router>
</config>
```
- Actually create action class which will be the target class of your router. It shoul extend `Magento\Framework\App\Action\Action` (as other Magento action classes). 

So stages of routing in Magento 2:
- `Magento/Framework/App/RouterList` collects list of all routers.
- `Magento/Framework/App/FrontController` searches through a list of routers according to their sort order and call method `match()`
- Method `match` in router returns either `null` or sets Module, Controller and Action to request and returns action instance.
- If action is returned, Front Controller starts to process request. During processing method execute of Action instance is called.
- If Base Router return null, Cms Router is called and sets proper Module-Controller-Action. 
- If Cms Router returns null, Default Router forwards to `\Magento\Cms\Controller\Noroute\Index` which returns 404 page (using `noRouteHandlerList`). Default router is last magento router to check.

**NOTICE:** 
- Custom router should return `\Magento\Framework\App\Action\Forward`, not concrete action instance.
- FrontConroller looks for router untill $request->isDispatched() == false
- `\Magento\Framework\App\Action\Forward::execute()` sets $request->setDispatched(false)
- So if your custom router has sort order lower, than base router **you get an infinite loop**. Custom router returns `Forward` action, `Forward` is executed and sets `dispatched=false`, and you never reach Base Router which actually resolves route.
- To avoid this you should:
  - Either set your custom router sort order greater than Base router sort order (i.e. > 30)
  - Or return null instead of Forward action in you custom router if request params are set properly.

In `routes.xml` it's possible to add `before` and `after` params to route which allows to override or extend existing routes.
```xml
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:App/etc/routes.xsd">
    <router id="standard">
        <route id="customer">
            <module name="OrangeCompany_RoutingExample" before="Magento_Customer" />
        </route>
    </router>
</config>
```
