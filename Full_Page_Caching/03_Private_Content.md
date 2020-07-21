**Private content**
(https://devdocs.magento.com/guides/v2.2/extension-dev-guide/cache/page-caching/private-content.html)

`customer-data.js` library is used for private data storage.
Steps to add private content on cacheable page:
- Create section source class which implements `Magento\Customer\CustomerData\SectionSourceInterface` and add it to SectionPoolInterface using `di.xml`:
```xml
<type name="Magento\Customer\CustomerData\SectionPoolInterface">
    <arguments>
        <argument name="sectionSourceMap" xsi:type="array">
            <item name="custom-name" xsi:type="string">Vendor\ModuleName\CustomerData\ClassName</item>
        </argument>
    </arguments>
</type>
```
Notice that this call should be under CustomerData folder.

- Create block and template. Initialize 