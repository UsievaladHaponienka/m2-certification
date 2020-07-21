#15 Creating UI component listing:
###1.Create table in database (declarative schema + whitelist). Create CRUD (Model, Resource Model, Collection at least)

###2.Create adminhtml/routes.xml, entry point (for example, menu.xml), controller and layout.

###3.Add UI component in layout:
```xml
    <body>
        <referenceContainer name="content">
            <uiComponent name="study_ui_listing"/>
        </referenceContainer>
    </body>
```
###4.Create UI component:

####4.1 Create listing component and declare its data source:
```xml
<listing xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:module:Magento_Ui:etc/ui_configuration.xsd">
    <argument name="data" xsi:type="array">
        <item name="js_config" xsi:type="array">
            <item name="provider" xsi:type="string">study_ui_listing.study_ui_listing_data_source</item>
        </item>
    </argument>
</listing>
```
*Variables:* provider, something like data provider "full name" consists of `<ui_component_file_name>`.`<data_source_name>` (see below).

####4.2 Configure Data Source:
```xml
<listing>
    <dataSource name="study_ui_listing_data_source" component="Magento_Ui/js/grid/provider">
        <settings>
            <updateUrl path="mui/index/render"/>
        </settings>
        <aclResource>Study_Ui::study_ui_base</aclResource>
        <dataProvider class="Magento\Framework\View\Element\UiComponent\DataProvider\DataProvider" name="study_ui_listing_data_source">
            <settings>
                <requestFieldName>entity_id</requestFieldName>
                <primaryFieldName>entity_id</primaryFieldName>
            </settings>
        </dataProvider>
    </dataSource>
</listing>
```
*Variables:* 
 - dataSource name: name of data source, `<ui_component_file_name>_data_source`.
 - dataSource component **can** be custom.
 - dataProvider name: `<ui_component_file_name>_data_source`.
 - dataProvider class **can** be custom.
 - aclResource: real acl resource.
 - dataProvider settings primaryFieldName: primary key in database table
 - dataProvider settings requestFieldName: -//-

####4.3 Finish listing configuration:
```xml
    <settings>
        <spinner>study_ui_columns</spinner>
        <deps>
            <dep>study_ui_listing.study_ui_listing_data_source</dep>
        </deps>
    </settings>
```
*Variables:* 
 - spinner: name of columns component (see below).
 - deps - dependency on component initialization, should be data provider "full name": `<ui_component_file_name>`.`<data_source_name>`.

####4.4 Define data source in `di.xml` **(NOT IN `adminhtml/di.xml`!)**:

#####4.4.1 Define collection virtual type (based on SearchResult):
```xml
    <virtualType name="Study\Ui\Model\ResourceModel\Entity\Collection\Virtual" type="Magento\Framework\View\Element\UiComponent\DataProvider\SearchResult">
        <arguments>
            <argument name="mainTable" xsi:type="const">Study\Ui\Model\ResourceModel\Entity::TABLE_NAME</argument>
            <argument name="resourceModel" xsi:type="string">\Study\Ui\Model\ResourceModel\Entity</argument>
        </arguments>
    </virtualType>
```
*Variables:* 
 - virtualType name: name of virtual type, seems to be not strict.
 - mainTable: Main table with data for UI listing, can be string instead of const (created at stage 1).
 - resourceModel: Resource model which works with this table (also created at stage 1).

#####4.4.2 Add collection to `DataProvider\CollectionFactory`:
```xml
    <type name="Magento\Framework\View\Element\UiComponent\DataProvider\CollectionFactory">
        <arguments>
            <argument name="collections" xsi:type="array">
                <item name="study_ui_listing_data_source" xsi:type="string">Study\Ui\Model\ResourceModel\Entity\Collection\Virtual</item>
            </argument>
        </arguments>
    </type>
```
*Variables:* 
 - item name: data source name.
 - item value: collection virtual type class from previous stage.
 
####4.5 Create columns component:
```xml
<listing>
    <columns name="study_ui_columns">
    </columns>
</listing>
```
*Variables:* 
 - columns name: columns name. Should be same as `<spinner>` in 4.3.
 
####4.6 Configure column components:
```xml
<columns>
    <column name="entity_id" sortOrder="0">
        <settings>
            <label>Id</label>
        </settings>
    </column>
</columns>
```
Notice, that label is **required**.

This id enough for basic UI grid. Improvements:

###1.Add selection column:
```xml
<columns>
    <selectionsColumn name="ids" sortOrder="0">
        <settings>
            <indexField>entity_id</indexField>
        </settings>
    </selectionsColumn>
</columns>
```
*Variables:* 
 - indexField: table primary key. 

###2.Add filtering:
 
####2.1 Configure filtering fro columns:
```xml
    <column name="last_name" sortOrder="30">
        <settings>
            <filter>text</filter>
        </settings>
    </column>
```
*Variables:* 
 - filter: filter type.

####2.2 Add listing toolbar:
```xml
<listing>
    <listingToolbar name="listing_top">
        <settings>
            <sticky>true</sticky>
        </settings>
        <bookmark name="bookmarks"/>
        <columnsControls name="columns_controls"/>
        <exportButton name="export_button"/>
        <filterSearch name="fulltext"/>
        <filters name="listing_filters"/>
        <paging name="listing_paging"/>
    </listingToolbar>
</listing>
```

Each node adds corresponding component to listing (seems more or less obvious).

####2.3 Add Mass action:
```xml
<listingToolbar>
    <massaction name="listing_massaction" component="Magento_Ui/js/grid/tree-massactions">
        <action name="delete">
            <settings>
                <confirm>
                    <message translate="true">Are you sure you want to delete the selected entities?</message>
                    <title translate="true">Delete entities</title>
                </confirm>
                <url path="study_ui/index/massDelete"/>
                <type>delete</type>
                <label translate="true">Delete</label>
            </settings>
        </action>
    </massaction>
</listingToolbar>
```

*Variables:* 
 - action name: name of action
 - settings confirm: action confirmation settings.
 - settings confirm message & settings config title: confirmation message text.
 - url path: route to action controller

Controller's method `execute()` can look like this:
```php
public function execute()
{
if ($this->getRequest()->getParam('selected')) {
    foreach ($this->getRequest()->getParam('selected') as $id) {
        $this->entityRepository->deleteById((int)$id);
    }
}

return $this->_redirect('study_ui/index/index');
}
```

####2.4 Add action columns
```xml
<columns>
        <actionsColumn name="actions" class="Study\Ui\Ui\Component\Listing\Column\Actions">
            <settings>
                <indexField>entity_id</indexField>
            </settings>
        </actionsColumn>
</columns>
```

Variables:* 
 - actionsColumn class: Class responsible for actions options
 
Class should look like this:
```php
<?php
declare(strict_types=1);

namespace Study\Ui\Ui\Component\Listing\Column;

use Magento\Framework\UrlInterface;
use Magento\Framework\View\Element\UiComponent\ContextInterface;
use Magento\Framework\View\Element\UiComponentFactory;
use Magento\Ui\Component\Listing\Columns\Column;

class Actions extends Column
{
    /**
     * @var UrlInterface
     */
    private $urlBuilder;

    /**
     * @param ContextInterface $context
     * @param UiComponentFactory $uiComponentFactory
     * @param UrlInterface $urlBuilder
     * @param array $components
     * @param array $data
     */
    public function __construct(
        ContextInterface $context,
        UiComponentFactory $uiComponentFactory,
        UrlInterface $urlBuilder,
        array $components = [],
        array $data = []
    ) {
        $this->urlBuilder = $urlBuilder;
        parent::__construct($context, $uiComponentFactory, $components, $data);
    }

    /**
     * @inheritDoc
     */
    public function prepareDataSource(array $dataSource)
    {
        if (isset($dataSource['data']['items'])) {
            $storeId = $this->context->getFilterParam('store_id');
            foreach ($dataSource['data']['items'] as &$item) {
                $item[$this->getData('name')]['edit'] = [
                    'href' => $this->urlBuilder->getUrl(
                        'study_ui/index/edit',
                        ['id' => $item['entity_id'], 'store' => $storeId]
                    ),
                    'label' => __('Edit'),
                    'hidden' => false
                ];
            }
        }

        return $dataSource;
    }
}
```

####2.5 Add inline editor
#####2.5.1 Add `editor` component to `column`
```xml
<column>
    <settings>
        <editor>
            <editorType>text</editorType>
        </editor>
    </settings>
</column>
```

#####2.5.2 Add editor to `columns` component
```xml
<columns>
    <editorConfig>
        <param name="clientConfig" xsi:type="array">
            <item name="saveUrl" xsi:type="url" path="study_ui/index/inlineEdit"/>
            <item name="validateBeforeSave" xsi:type="boolean">false</item>
        </param>
        <param name="indexField" xsi:type="string">entity_id</param>
        <param name="enabled" xsi:type="boolean">true</param>
        <param name="selectProvider" xsi:type="string">study_ui_listing.study_ui_listing.study_ui_columns.ids</param>
    </editorConfig>
    <childDefaults>
        <param name="fieldAction" xsi:type="array">
            <item name="provider" xsi:type="string">study_ui_listing.study_ui_listing.study_ui_columns_editor</item>
            <item name="target" xsi:type="string">startEdit</item>
            <item name="params" xsi:type="array">
                <item name="0" xsi:type="string">${ $.$data.rowIndex }</item>
                <item name="1" xsi:type="boolean">true</item>
            </item>
        </param>
    </childDefaults>
</columns>
```
Variables:* 
 - saveUrl: Path to inline controller edit (see below)
 - editorConfig selectProvider: `<ui_component_file_name>`.`<ui_component_file_name>`.`<columns_compoent_name>`.`<selection_columns_component_name>`.
 - childDefaults provider: `<ui_component_file_name>`.`<ui_component_file_name>`.`<columns_compoent_name>_editor`.
 
#####2.5.3 Create inline edit controller
```php
<?php
declare(strict_types=1);

namespace Study\Ui\Controller\Adminhtml\Index;

use Magento\Backend\App\Action;
use Magento\Backend\App\Action\Context;
use Magento\Framework\Controller\Result\JsonFactory;
use Study\Ui\Api\EntityRepositoryInterface;

class InlineEdit extends Action
{
    /**
     * @var JsonFactory
     */
    private $jsonFactory;
    
    /**
     * @var EntityRepositoryInterface
     */
    private $entityRepository;

    /**
     * @param Context $context
     * @param JsonFactory $jsonFactory
     * @param EntityRepositoryInterface $entityRepository
     */
    public function __construct(
        Context $context,
        JsonFactory $jsonFactory,
        EntityRepositoryInterface $entityRepository

    ) {
        parent::__construct($context);
        $this->jsonFactory = $jsonFactory;
        $this->entityRepository = $entityRepository;
    }

    /**
     * @inheritDoc
     */
    public function execute()
    {
        $resultJson = $this->jsonFactory->create();
        $error = false;
        $messages = [];

        if ($this->getRequest()->getParam('isAjax')) {
            $items = $this->getRequest()->getParam('items', []);
            if (!count($items)) {
                $messages[] = __('Incorrect data');
                $error = true;
            } else {
                foreach ($items as $itemId => $value) {
                    try {
                        $entity = $this->entityRepository->getById($itemId);
                        $entity->setData(array_merge($entity->getData(), $items[$itemId]));
                        $this->entityRepository->save($entity);
                    } catch (\Exception $e) {
                        $messages[] = __($e->getMessage());
                        $error = true;
                    }

                }
            }
        }

        return $resultJson->setData([
            'messages' => $messages,
            'error' => $error
        ]);
    }
}
```

####2.6 Add 'Add new' button:
#####2.6.1 Add button to listing settings:
```xml
<listing>
    <settings>
        <buttons>
            <button name="add">
                <url path="*/*/new"/>
                <class>primary</class>
                <label translate="true">Add New Entity</label>
            </button>
        </buttons>
    </settings>
</listing>
```

Variables:* 
 - label: Button title.
 - url path: path to controller (see below).
 - class: button css class.
#####2.6.2 Add controller to handle request
Pay attention that `New` is not allowed name for Controller class, But `NewAction` is OK.
Controller may look like this:
```php
<?php
declare(strict_types=1);

namespace Study\Ui\Controller\Adminhtml\Index;

use Magento\Framework\App\Action\HttpGetActionInterface;
use Magento\Customer\Controller\Adminhtml\Index;

class NewAction extends Index implements HttpGetActionInterface
{
    public function execute()
    {
        $resultForward = $this->resultForwardFactory->create();
        $resultForward->forward('edit');
        return $resultForward;
    }
}
```

Notice that corresponding layout and `form` component is required (see manual about form).
