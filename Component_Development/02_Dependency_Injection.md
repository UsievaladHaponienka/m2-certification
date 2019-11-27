**2. Dependency Injection (DI)** (https://devdocs.magento.com/guides/v2.3/extension-dev-guide/depend-inj.html)

DI is a pattern which allows object A to declare its dependensies to object B. Object B is usually an Interface and Object B decides, which implementation should be provided to object A (based on config).

Automatic DI in Magento 2 allows to initialize object A without manual initialization of object B, which can also can have own dependences, which have their own and so on.

Using Interfaces in DI is better than using concrete implementations. It decreases risk of incompatibility, because Magento implementation can change while interfece remains the same. Also it allows to use preference for interface instead of rewriting class.

Object Manager is Magento service class for object creation. Object Manager is configured in di.xml.

Object Manager injects all dependences required for object. Concrete implementations are resolved by Code Compiler Tool.
Code compiler Tool also creates generated classes - Factories, Interceptors and Proxy.

Class SHOUD NOT depend on Object Manager. Exceptions are:
- Custom Factories and Proxies.
- Integration tests.
- For backward compatibility.
- For usage of Object Manager static magic methods like `__sleep()`, `__wakeup()`, etc.

Injections can be:
- Constructor
- Method
In constructor injection dependendy is declared in class `__construct` method:
```php
public function __construct(
    Magento\Backend\Model\Menu\Item\Factory $menuItemFactory,  // Service dependency
) {
    $this->_itemFactory = $menuItemFactory;
}
```
In method injection dependency is passed to method. When an object needs to perform actions on a dependency that cannot be injected, use method injection:
```php
public function processCommand(\Magento\Backend\Model\Menu\Builder\AbstractCommand $command) // API param
```
Object types by DI pattern can be:
- Injectable
- Newable/Non-injectable

Injectable objects are SINGLETON service classes.
Non-injectable objects can not be injected because they require additional data (for example, product_id for Product) for their initializtion. To use such type of object, ObjectFactory in neccessary.
