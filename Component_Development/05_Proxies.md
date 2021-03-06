**5. Proxies** (https://devdocs.magento.com/guides/v2.3/extension-dev-guide/proxies.html)

Constructor injection causes chain reaction of objects instantiations, because all objects, decalred as dependency must be created, and all their dependencies, and so on. 
This can decrease application perfomance. Proxy classes are solution for this case.

Proxy classes are generated automatically and can be added using argument replacement in di.xml:
```xml
<type name="FastLoading">
    <arguments>
        <argument name="slowLoading" xsi:type="object">SlowLoading\Proxy</argument>
    </arguments>
</type>
```
Proxy classes (like Factories) depend on Object Manager and instantiated fast. Example of methods Proxy class (`\Magento\Translation\Model\Source\InitialTranslationSource\Proxy`):
```php
public function __construct(\Magento\Framework\ObjectManagerInterface $objectManager, $instanceName = <ClassName>, $shared = true)
{
    $this->_objectManager = $objectManager;
    $this->_instanceName = $instanceName;
    $this->_isShared = $shared;
}

public function get($path = '')
{
    return $this->_getSubject()->get($path);
}

protected function _getSubject()
{
    if (!$this->_subject) {
        $this->_subject = true === $this->_isShared
            ? $this->_objectManager->get($this->_instanceName)
            : $this->_objectManager->create($this->_instanceName);
    }
    return $this->_subject;
}
```
So class instance is only created when method `get()` of the original class is called.
