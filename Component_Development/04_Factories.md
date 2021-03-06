**4. Factories** (https://devdocs.magento.com/guides/v2.3/extension-dev-guide/factories.html)

Factories are service classes which allows to get instance of non-injectable object (i.e. object which can not be initialized without additional data).

Object Manager has two main methods - `get()` and `create()`. When new object is instantiated `get()` method is used by default:
```php
public function get($type)
{
    $type = ltrim($type, '\\');
    $type = $this->_config->getPreference($type);
    if (!isset($this->_sharedInstances[$type])) {
        $this->_sharedInstances[$type] = $this->_factory->create($type);
    }
    return $this->_sharedInstances[$type];
}
```

So Object Manager actually creates every object only once (by default), next instantiations of some object will return **existent one**.
In Factory method create() Object Manager create() is called:
```php
public function create(array $data = [])
{
    return $this->_objectManager->create($this->_instanceName, $data);
}
``` 

So Factory forces Object Manager to create new instance instead of returning existent one.
In addition, Factories depend only on Object Maneger. So it's impossible to create non-injectable object without passing certain params to its constructor, 
but possible to create factory without any params and pass this params to Factory create() method later.

Factories are generated automaticaly, but since `generated/` folder has lover priority, than `vendor/` or `app/` it is possible to create custom factory. It is neccessary *only* if specific behavior of factory is required.

Factories CAN resolve interface dependences and preferences. So if you:
- Configure preference `Class` for interface `Interface`,
- inject InterfaceFactory in your constructor,

you will get ClassFactory instance.
