**8. Plugins** (https://devdocs.magento.com/guides/v2.3/extension-dev-guide/plugins.html)

Plugin or Interceptor is a class that modifies behavior by intercepting function call and running code before, around or after call.
Plugins CAN NOT be appiled to:
- Final methods and final classes (Because interceptors extend observed classes and re-define their methods, final classes can not be extended, final methods can not be re-defined)
- Private or protected methods
- Static methods
- `__construct()` (but can be aplied to `_construct()` with one underscore)
- Virtual types
- Objects that are instantiated before Magento\Framework\Interception is bootstrapped

**Typical function in Interceptor class:**
```php
public function getTableName($modelEntity, $connectionName = 'default')
{
    $pluginInfo = $this->pluginList->getNext($this->subjectType, 'getTableName'); 	// subjectType is set in __init function of trait \Magento\Framework\Interception\Interceptor and contains target class name
    											// So in this line next plugin for method getTableName of class ctored in $this->subjectType is found
    if (!$pluginInfo) {
        return parent::getTableName($modelEntity, $connectionName);			// If no plugins were found, observed method is called
    } else {
        return $this->___callPlugins('getTableName', func_get_args(), $pluginInfo);	//Plugin method is called
    }
}
```
**Plugins must be defined in di.xml.**
```xml
<config>
    <type name="{ObservedType}">
    	<plugin name="{pluginName}" type="{PluginClassName}" sortOrder="1" disabled="false" />
    </type>
</config>
```
**Plugins can be before, after and around.**

__Before plugins' methods:__
- Modify observed method's arguments
- Executed before call of observed method
- Must have name `before<ObservedMethodName>`
- Accept subject (observed method class) + all arguments of observed method (in corresponding order)
- Return argument, array of arguments (if there are more than one) of null (if arguments weren't changed)
```php
public function beforeSetName(\Magento\Catalog\Model\Product $subject, $name)
{
    return ['(' . $name . ')'];
}
```
__After plugins' methods:__
- Modify observed method's return value
- Executed after call of observed method
- Must have name `after<ObservedMethodName>`
- Accept subject (observed method class) and result. 
- Also can accept observed methods arguments whish should passed after result in corresponding order(it's not neccessary to pass all arguments, only those before required)
- If observed method returns void, result=null. As well as result=null for next after-plugins if plugin method returns void
- If an argument is optional in the observed method, then the after method should also declare it as optional.

__Around plugins' methods:__
- Can totaly change method behavior
- Executed before and after observed method
- Must have name `around<ObservedMethodName>`
- Accept subject, callable (observed method) and MUST accept all arguments of observed method (as before plugins)
- When around plugin executes callabe Magento runs NEXT PLUGIN or observed method if there are no next plugins.
- Return result (as after plugins)

**Plugin execution order rules:**
1. Before the execution of observed methods plugins' methods are executed from LOWEST TO GREATEST sort order
2. First of all Magento executes `before` methods.
3. After `before` methods executed, `around` method of CURRENT PLUGIN is called (not next plugins' `before` method)
4. After observed method execution plugins' methods are executed from GREATEST TO LOWEST sort order
5. Magento executes second part of `around` methods.
6. After second part of `around` method is executed, `after` method of CURRENT PLUGIN us called (not next around).

**Example:**
- Plugin A, methods `before`, `around`, `after`, sort order=10
- Plugin B, methods `before`, `around`, `after`, sort order=20

Execution order:

- A-before *(before method of plugin with lowest sort order)*
- A-around-first-part *(A-before finished, around method of current plugin is called)*
- B-before *(proceed called, so next plugin before method executed)*
- B-around-first-part *(B-before finished, around method of current plugin is called)*
- Proceed
- B-around-second-part *(second part of around method of plugin with greates sort order)*
- B-after *(B-around-second-part finished, after method of currn plugin is called)*
- A-around-second-part *(B-after finished, next plugin is called)*
- A-after *(A-around-second-part finished, after method of currn plugin is called)*

Plugins also work for classes which extend/implement target class.
To disable plugin you need to add disabled="true" attribute to `<plugin>` node.