**10. Indexing** (https://devdocs.magento.com/guides/v2.3/extension-dev-guide/indexing.html)

The goal of indexing is to improve storefront perfomance. Magento keeps a lot of merchant data (like catalog data, prices, users, stores) in many database tables. This allows to store, for example, final product price (with all priice rules, discounts, etc) in separate table and get data directly from this table (instead of re-calculating it everytime) while loading product page.

Indexing Terminilody:
- *Dictionary.* Original data entered to the system. Dictionaries are organized in normal form (https://en.wikipedia.org/wiki/Database_normalization)
- *Index.* Representation of original data. Can contain results of aggregations or calculations. Algorith for recreating index data from original data is required.
- *Indexer.* Object that creates index.

Magento components used for indexing:
- Module `Magento_Indexer`:
  - Indexer declaration.
  - Indexer running.
  - Indexer running mode config.
  - Indexer status.
- `Magento\Framework\Mview`:
  - Allows to track DB changes of certain entity and run change handler.
  - Emulates materialized view for MySQL (https://en.wikipedia.org/wiki/Materialized_view). Thois allows to get data executing php-code instead of direct SQL-queries.

Each index allows *full reindex* and *partial reindex*. 
- Full reindex crebuilds all indexing-related data (for example, after creating new store).
- Partial reindex rebuilds database only for tables only for changed entities.

Dependency "Type of changes made to original data" -> "Type of reindex required" is specific for each indexer.

Indexing flow:
- Dictionary is changed.
- If indexer configured as update on save, `Magento_Indexer` updates index tables.
- If indexer configured as update on schedule,`Magento\Framework\Mview` add ids of changed data to changelof table and changes index status to `Invalid`.
- `Magento_Indexer` updates index tables related to changelog table.

`mview.xml` file is used to track database changes of certain entity.
```xml
<view id="catalog_category_product" class="Magento\Catalog\Model\Indexer\Category\Product" group="indexer">
  <subscriptions>
    <table name="catalog_category_entity" entity_column="entity_id" />
    <table name="catalog_category_entity_int" entity_column="entity_id" />
  </subscriptions>
</view>
```

- `view` node defines indexer. `id` - name of indexer table, `class` - index executor, `group` - indexer group.
- `subscriptions` node is list ofr tables to track/
- `table` node defines table to track. `entity_column` - primary column of entity. If something changes in this tables (new categories added, categories deleted) method `Magento\Catalog\Model\Indexer\Category\Product:;execute()` is called with ids of entities to reindex.

Changelog table (table which contains data about entites to reindex on schedule) has name like `<tracked_table>_cl`. Method `Magento\Framework\Mview\ViewInterface::update()` (called by cron) is responsible for handling records in changelog.

You can reindex data:
- By running cron job
- By running CLI-command `bin/magento indexer:reindex`

**Adding custom indexer**

Custome Indexer class must implement `\Magento\Framework\Indexer\ActionInterface` and provide following operations:
- Row reindex: processing single entry in database.
- List reindex: process set of entries.
- Full reindex: process all entities from dictionary.

*To add custom index:*

- Define index in `etc/indexer.xml`:
```xml
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:Indexer/etc/indexer.xsd">
    <indexer id="merchandizing_popular" view_id="merchandizing_popular_order" class="Vendor\Merchandizing\Model\Indexer\Popular">
        <title translate="true">Popular Products</title>
        <description translate="true">Sort products in a category by popularity</description>
    </indexer>
</config>
```

- Add new `mview.xml`:
```xml
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:Mview/etc/mview.xsd">
    <view id="merchandizing_popular_order" class="Vendor\Merchandizing\Model\Indexer\Popular" group="indexer">
        <subscriptions>
            <table name="sales_order" entity_column="entity_id" />
        </subscriptions>
    </view>
</config>
```

- Create indexer Class which implements `\Magento\Framework\Indexer\ActionInterface` and `\Magento\Framework\Mview\ActionInterface`:
```php
class Popular implements \Magento\Framework\Indexer\ActionInterface, \Magento\Framework\Mview\ActionInterface
{
    /*
     * Used by mview, allows process indexer in the "Update on schedule" mode
     */
    public function execute($ids){
        //Used by mview, allows you to process multiple placed orders in the "Update on schedule" mode
    }

    public function executeFull(){
        //Should take into account all placed orders in the system
    }

    public function executeList(array $ids){
        //Works with a set of placed orders (mass actions and so on)
    }

    public function executeRow($id){
        //Works in runtime for a single order using plugins
    }
}
```
