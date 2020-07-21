#1. di.xml file
(https://devdocs.magento.com/guides/v2.2/extension-dev-guide/build/di-xml-file.html)
The main goal of `di.xml` is to configure dependencies which are injected by object manager. Also sensitive data can be specified here.

`di.xml` are merged by appending all nodes. Area specific files (i.e. etc/frontend/di.xml and etc/adminhtml/di.xml) have grater priority. As a general rule, the area specific `di.xml` files should configure dependencies for the presentation layer, and your module’s global `di.xml` file should configure the remaining dependencies.

##Type configurations
Type configurations describe an object lifestyle and how to instantiate it.

```xml
<virtualType name="moduleConfig" type="Magento\Core\Model\Config">
    <arguments>
        <argument name="type" xsi:type="string">system</argument>
    </arguments>
</virtualType>
<type name="Magento\Core\Model\App">
    <arguments>
        <argument name="config" xsi:type="object">moduleConfig</argument>
    </arguments>
</type>
```

- `moduleConfig` - a *virtual type* that extends `Magento\Core\Model\Config`.
- `Magento\Core\Model\App` - all instances of this type receive an instance of `moduleConfig` as dependency.

###Virtual types
A virtual type allows to change the arguments of specific dependency and change the behavior of a particular class. This allows to use customized class without affecting other classes which depend on original.

##Constructor arguments
Constructor arguments are declared in `arguments` node. The object manager injects this arguments during class creation.
The `name` of argument must correspond to the parameter in the constructor:
```xml
<type name="Magento\Core\Model\Session">
    <arguments>
        <argument name="sessionName" xsi:type="string">adminhtml</argument>
    </arguments>
</type>
```

This example creates instances of Magento\Core\Model\Session with the class constructor argument $sessionName set to a value of adminhtml.

###Argument types

Declaration: `<argument xsi:type={type}>{value}</argument>`

Types:
- object. `{value}` is class full class name. Attributes:
  - shared, true/false. If shared="false" each injection creates new instance. In other words, shared="false" forces object manager to use `create()` method instead of `get()` during this injection.
- string. `{value}` is interpreted as string. Attributes:
  - translate, true/false.
- boolean. `{value}` is interpreted as boolean value. So `1` or `"1"` is interpreted as `true`.
- number. `{value}` is integer, float or numeric string.
- init_parameter. `{value}` is `Constant::NAME`
- const. `{value}` is same as in init_parameter.
- null. Null value, no `{value}` in node is required. 
- array. Has sub-nodes `item`:
```xml
<item name="someKey" xsi:type="<type>">someVal</item> <!-- someKey - key in array, someVal - corresponding value -->
```

Example:
```xml
<type name="Magento\Example\Type">
    <arguments>
        <!-- Pass simple string -->
        <argument name="stringParam" xsi:type="string">someStringValue</argument>
        <!-- Pass instance of Magento\Some\Type -->
        <argument name="instanceParam" xsi:type="object">Magento\Some\Type</argument>
        <!-- Pass true -->
        <argument name="boolParam" xsi:type="boolean">1</argument>
        <!-- Pass 1 -->
        <argument name="intParam" xsi:type="number">1</argument>
        <!-- Pass application init argument, named by constant value -->
        <argument name="globalInitParam" xsi:type="init_parameter">Magento\Some\Class::SOME_CONSTANT</argument>
        <!-- Pass constant value -->
        <argument name="constantParam" xsi:type="const">Magento\Some\Class::SOME_CONSTANT</argument>
        <!-- Pass null value -->
        <argument name="optionalParam" xsi:type="null"/>
        <!-- Pass array -->
        <argument name="arrayParam" xsi:type="array">
            <!-- First element is value of constant -->
            <item name="firstElem" xsi:type="const">Magento\Some\Class::SOME_CONSTANT</item>
            <!-- Second element is null -->
            <item name="secondElem" xsi:type="null"/>
            <!-- Third element is a subarray -->
            <item name="thirdElem" xsi:type="array">
                <!-- Subarray contains scalar value -->
                <item name="scalarValue" xsi:type="string">ScalarValue</item>
                <!-- and application init argument -->
                <item name="globalArgument " xsi:type="init_parameter">Magento\Some\Class::SOME_CONSTANT</item>
            </item>
        </argument>
    </arguments>
</type>
```

###Abstraction-implementation mappings

This mappings are used when constructor required interface. Mappings are used to determine necessary implementation of this interface.
`Preference` node defines default implementation:
```xml
<config>
    <preference for="Magento\Core\Model\UrlInterface" type="Magento\Core\Model\Url" />
</config>
```

###Object lifestyle configuration

