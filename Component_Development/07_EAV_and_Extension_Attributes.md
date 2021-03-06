**6. EAV and Extension Attributes** (https://devdocs.magento.com/guides/v2.3/extension-dev-guide/code-generation.html, 
https://devdocs.magento.com/guides/v2.3/extension-dev-guide/extension_attributes/adding-attributes.html)

- Custom and EAV (Entity Attribute Value) attributes are those which can be added via admin panel. Custom attributes are subset of EAV attributes.
- Extension attributes are used to exten functionality and can use more complex data types than custom attribute. Thus attributes do not apper in admin panel.

EAV attributes can be added programmatiocaly by Data Patch(`\Magento\Customer\Setup\Patch\Data\AddCustomerUpdatedAtAttribute`):
```php
public function apply()
{
    $customerSetup = $this->customerSetupFactory->create(['setup' => $this->moduleDataSetup]);
    $customerSetup->addAttribute(
    Customer::ENTITY, //entity type id, 'customer' in this case
     'attribute_code', [
            'type' => 'static',
            'label' => 'Updated At',
            'input' => 'date',
            'required' => false,
            'sort_order' => 87,
            'visible' => false,
            'system' => false,
    ]);
}
```
Extension attributes are declared at etc/extension_attributes.xml:
```xml
<extension_attributes for="Path\To\Interface">
    <attribute code="name_of_attribute" type="datatype">
       <resources>
          <resource ref="permission"/>
       </resources>
       <join reference_table="" reference_field="" join_on_field="">
          <field>fieldname</field>
       </join>
    </attribute>
</extension_attributes>
```

- for: path to class which processes extensions. This class must implement ExtensibleDataInterface
- type: can be simple (like string or integer) or comlex, like interface
- ref: name of acl resource
- field: property if interface (used in 'type')

Join directive is neccessary to make collection filterable by attruibute.

To add extension attribute to entity first of all it's neccessary to create after plugin for entity repository:
```php
public function afterGet
(
    \Magento\Catalog\Api\ProductRepositoryInterface $subject,
    \Magento\Catalog\Api\Data\ProductInterface $entity
) {
    $ourCustomData = $this->customDataRepository->get($entity->getId());

    $extensionAttributes = $entity->getExtensionAttributes(); /** get current extension attributes from entity **/
    $extensionAttributes->setOurCustomData($ourCustomData);
    $entity->setExtensionAttributes($extensionAttributes);

    return $entity;
}
```

In this case we:
- Get extension attibutes if they are already set
- Add our extension attribute
- set updated etension attributes to entity

If entity doesn't have implementation of extension attributes, getExtensionAttributes() method returns null. 
In this case it's neccessary to create plugin for entity data interface (not only for repository).
```PHP
public function afterGetExtensionAttributes(
    ProductInterface $entity,
    ProductExtensionInterface $extension = null
) {
    if ($extension === null) {
        $extension = $this->extensionFactory->create();
    }

    return $extension;
}
```
So, in general, to add extension attribute to Entity, we:
- Usually need to create CRUD-like system (to work with table/CRM/any other source of our extension attribute)
- Define extension attribute in `extension_attributes.xml` for EntityDataInterface
- Create after plugins for EntityRepository which add extension attribute on `getList()` and `getById()` methods.
