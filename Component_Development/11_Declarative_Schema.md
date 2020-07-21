**Declarative schema** (https://devdocs.magento.com/guides/v2.3/extension-dev-guide/declarative-schema/)

Declarative schema allows to declare final desired state of database instead of using several update scripts which are executed one by one according to version. This approach simplifies installation and upgrade process for extension developers.

Previously:
- *InstallData* and *InstallSchema* scripts, which are executed the first time a module is installed.
- *UpgradeData* and *UpgradeSchema* incremental scripts, which supplement an existing module schema.
- *Recurring* scripts, which are executed each time you install or upgrade Magento.

Terminology:
- *Data patch* - class that contains data modification instructions. Can be dependent on other patches.
- *Revertable data patch* - Data patch that contains `revert()` method.
- *Migration* - Type of non-revertable data patch which can be applied, but not reverted.
- *Schema patch* - class that conttains schema modifocation instructions. Allows:
  - Adding triggers, stored procedures, functions
  - Performing data migration with inside DDL operations
  - Renaming tables, columns, and other entities
  - Adding partitions and options to a table
- *Revertable schema patch* - Shema patch with `revert()` functionality

**Old (pre Magento 2.3.0) scripts migration**

Usefull cli commands:

- `bin/magento setup:db-declaration:generate-patch [options] <module-name> <patch-name>` - generates a patch stub. Options can be:
  - `--revertable[=true | false]`
  - `--type[=<type>]` - type of patch to generate. Default is `data`

- `bin/magento setup:install` and `bin/magento setup:upgrade`:
  - `--convert-old-scripts=1` - converts install or upgrade scripts into declarative schema
All released modules that used Upgrade scripts must implement `Magento\Framework\Setup\Patch\PatchVersionInterface` and `getVersion()` method for backward compatibility. This method allows to skip changes made in previous version by old scripts. 

  - `--dry-run=1` - instead of upgrading database, Magento logs SQL statements in var/log/dry-run-installation.log. This allows you to check result of executing this commands withous actually executing them.
  - `--safe-mode=1` - creates a data dump for data that can be lost during installation/upgrade.
  - `--data-restore=1` - Only with `setup:upgrade`, rollback.

When safe mode is enabled, Magento creates .csv dump file every time destructive operation takes place (operations that lead to data loss). This dumps are located at ver/declarative_dumps_csv.

 - `bin/magento setup:db-declaration:generate-whitelist [options]` - generates `db_schema_whitelist.json`. `[options]` usually are `--module-name=Vendor_Module`.

In current state tables and columns can be modified both by db_schema and by setup/upgrade scripts and Magento doesn't know which tables/columns can be safely altered only using db_schema. The `db_schema_whitelist.json` file contains all DB elements added by db_schema and therefore can be safely altered by new version of `db_schema.xml`. It's recomended to regenerate whitelist for each release.

Whitelist **CAN NOT** be generated automaticaly if tables in database have prefixes, such whitelist should be generated manualy or on development environment without prefixes.

**Declarative schema example (from Magento_Catalog):**
```xml
<table name="catalog_product_entity_datetime" resource="default" engine="innodb"
           comment="Catalog Product Datetime Attribute Backend Table">
    <column xsi:type="int" name="value_id" padding="11" unsigned="false" nullable="false" identity="true" comment="Value ID"/>
    <column xsi:type="smallint" name="attribute_id" padding="5" unsigned="true" nullable="false" identity="false" default="0" comment="Attribute ID"/>
    <column xsi:type="smallint" name="store_id" padding="5" unsigned="true" nullable="false" identity="false" default="0" comment="Store ID"/>
    <column xsi:type="int" name="entity_id" padding="10" unsigned="true" nullable="false" identity="false" default="0" comment="Entity ID"/>
    <column xsi:type="datetime" name="value" on_update="false" nullable="true" comment="Value"/>
    <constraint xsi:type="primary" referenceId="PRIMARY">
        <column name="value_id"/>
    </constraint>
    <constraint xsi:type="foreign" referenceId="CAT_PRD_ENTT_DTIME_ATTR_ID_EAV_ATTR_ATTR_ID" table="catalog_product_entity_datetime" column="attribute_id" referenceTable="eav_attribute" referenceColumn="attribute_id" onDelete="CASCADE"/>
    <constraint xsi:type="foreign" referenceId="CAT_PRD_ENTT_DTIME_ENTT_ID_CAT_PRD_ENTT_ENTT_ID" table="catalog_product_entity_datetime" column="entity_id" referenceTable="catalog_product_entity" referenceColumn="entity_id" onDelete="CASCADE"/>
    <constraint xsi:type="foreign" referenceId="CATALOG_PRODUCT_ENTITY_DATETIME_STORE_ID_STORE_STORE_ID" table="catalog_product_entity_datetime" column="store_id" referenceTable="store" referenceColumn="store_id" onDelete="CASCADE"/>
    <constraint xsi:type="unique" referenceId="CATALOG_PRODUCT_ENTITY_DATETIME_ENTITY_ID_ATTRIBUTE_ID_STORE_ID">
        <column name="entity_id"/>
        <column name="attribute_id"/>
        <column name="store_id"/>
    </constraint>
    <index referenceId="CATALOG_PRODUCT_ENTITY_DATETIME_ATTRIBUTE_ID" indexType="btree">
        <column name="attribute_id"/>
    </index>
    <index referenceId="CATALOG_PRODUCT_ENTITY_DATETIME_STORE_ID" indexType="btree">
        <column name="store_id"/>
    </index>
</table>
```

- `<table> node`:
	- `name` - table name
	- `engine` - SQL engine, `innodb` or `memory`
	- `resource` - database shard on whcih to install table, '`default`, `checkout` or `sales`
	- `comment` - table comment

- `<column> node`:
	- `xsi:type` - column type:
	  - blob (includes mediumblob, longblob)
	  - boolean
	  - date
	  - datetime
	  - decimal
	  - float
	  - int (includes smallint, bigint, tinyint)
	  - real (includes decimalm float, double, real)
	  - text
	  - timestamp
	  - varbinary
	  - varchar
	- `default` - default column value
	- `disabled` - disables declared table, column, constraint or index
	- `identity` - indicates whether column is autoincremented
	- `length` - column lenght in test and binary (varchar, varbinary) types
	- `nullabe` - indicates whether column can be nullable
	- `onCreate` - trigger that allows to move data from one column to another on another column creation
	- `padding` - int column size
	- `precision` - number of allowed digits (before decimal) in real type
	- `scale` - number of digits after decimal in reql type
	- `unsigned` - for numeric types, indicates whether colum can contain both positive and negative values

- `<constraint> node>`:
	- `type` - `primary`, `unique` or `foreign`. `foreign` is simular to foreign key is SQL:
	  - table - name of current table
	  - column - column in CURRENT table
	  - referenceTable - table being referenced
	  - referenceColumn - column in REFERENCE table
	  - onDelete - can be `CASCADE`, `SET NULL` or `NO ACTION`
	- `referenceId` - custom identifier for relation mapping
Example:
```xml
<constraint xsi:type="primary" referenceId="PRIMARY">
    <column name="entity_id"/>
</constraint>
<constraint xsi:type="foreign" referenceId="COMPANY_CREDIT_COMPANY_ID_DIRECTORY_COUNTRY_COUNTRY_ID" table="company_credit" column="company_id" referenceTable="company" referenceColumn="entity_id" onDelete="CASCADE"/>
```
  - `<index> node`:
  	- `referenceId` - custom identifier for relation mapping
  	- `indexType` - can be `btree`, `fulltext` or `hash`
Example:
```xml
<index referenceId="NEWSLETTER_SUBSCRIBER_CUSTOMER_ID" indexType="btree">
    <column name="customer_id"/>
</index>
```

The following code fragment can be used to rename table (old table node removed, new table noe added):
```xml
- <table name="new_name">
+ <table name="new_name" onCreate="migrateDataFromAnotherTable(old_name)">
```

Same for column renaming:
```xml
- <column xsi:type="text" name="old_title" nullable="false" length="255" comment="Title"/>
+ <column xsi:type="text" name="new_title" nullable="false" length="255" comment="Title" onCreate="migrateDataFrom(old_title)"/>
```

When module is disabled, its database schema is not visible for upgrade/install. Practically, this means that if you disable a module which uses declarative schema and run bin/magento setup:upgrade, **its database tables will be dropped.** `setup:upgrade --safe-mode=1` should be used to create dump, `setup:upgrade --data-restore=1` to restore data after enabling module back.

**Data and schema patches**

Data patches are located in `<Vendor>/<Module_Name>/Setup/Patch/Data/<Patch_Name>.php` and must implement `\Magento\Framework\Setup\Patch\DataPatchInterface`. Patches are applied only once. List of applied patches is stored in `patch_list` table. Patches are applied by `setup:upgrade` command. To make patch revertable it must implement `Magento\Framework\Setup\Patch\PatchRevertableInterface`.

Patch sequence is handled through dependency-based approach. Dependencies are defined in `getDependencies()` method:
```php
public static function getDependencies()
{
    return [
        \SomeVendor\SomeModule\Setup\Patch\Data\SomePatch::class
    ];
}
```

Data patch full example:
```php
    namespace Magento\DummyModule\Setup\Patch\Data;
    use Magento\Framework\Setup\Patch\DataPatchInterface;
    use Magento\Framework\Setup\Patch\PatchRevertableInterface;

    class DummyPatch implements DataPatchInterface, PatchRevertableInterface
    {
        /**
         * @var \Magento\Framework\Setup\ModuleDataSetupInterface
         */
        private $moduleDataSetup;

        /**
         * @param \Magento\Framework\Setup\ModuleDataSetupInterface $moduleDataSetup
         */
        public function __construct(
            \Magento\Framework\Setup\ModuleDataSetupInterface $moduleDataSetup
        ) {
            /**
             * If before, we pass $setup as argument in install/upgrade function, from now we start
             * inject it with DI. If you want to use setup, you can inject it, with the same way as here
             */
            $this->moduleDataSetup = $moduleDataSetup;
        }

        /**
         * {@inheritdoc}
         */
        public function apply()
        {
            $this->moduleDataSetup->getConnection()->startSetup();
            //The code that you want apply in the patch
            //Please note, that one patch is responsible only for one setup version
            //So one UpgradeData can consist of few data patches
            $this->moduleDataSetup->getConnection()->endSetup();
        }

        /**
         * {@inheritdoc}
         */
        public static function getDependencies()
        {
            /**
             * This is dependency to another patch. Dependency should be applied first
             * One patch can have few dependencies
             * Patches do not have versions, so if in old approach with Install/Ugrade data scripts you used
             * versions, right now you need to point from patch with higher version to patch with lower version
             * But please, note, that some of your patches can be independent and can be installed in any sequence
             * So use dependencies only if this important for you
             */
            return [
                SomeDependency::class
            ];
        }

        public function revert()
        {
            $this->moduleDataSetup->getConnection()->startSetup();
            //Here should go code that will revert all operations from `apply` method
            //Please note, that some operations, like removing data from column, that is in role of foreign key reference
            //is dangerous, because it can trigger ON DELETE statement
            $this->moduleDataSetup->getConnection()->endSetup();
        }

        /**
         * {@inheritdoc}
         */
        public function getAliases()
        {
            /**
             * This internal Magento method, that means that some patches with time can change their names,
             * but changing name should not affect installation process, that's why if we will change name of the patch
             * we will add alias here
             */
            return [];
        }
    }
```

Example of `apply()` method:
```php
 public function apply()
        {
            $this->moduleDataSetup->getConnection()->startSetup();
            $sampleData = $this->csvReader->getData($fileName);	// Magento\Framework\File\Csv $scvReader, $fileName - path to csv file containing data
            $header = array_shift($sampleData);
	        foreach ($sampleData as $row) {			// I.e. for each entity, mentioned in csv file (row in file == entity)
	            $newEntityData = [];
	            foreach ($row as $key => $value) {		// Rearranging data from csv file
	                $newEntityData[$header[$key]] = $value;
	            }
	            $entity = $this->entityFactory->create();	// Setting data for new entity and saving entity
	            $entity->setData($newEntityData);
	            $this->entityRepository->save($entity);
	        }
            $this->moduleDataSetup->getConnection()->endSetup();
        }
```

Data pathes can be reverted by uninstalling module:
- `bin/magento module:uninstall Vendor_ModuleName` - for module installed by composer.
- `bin/magento module:uninstall --non-composer Vendor_ModuleName` - for module installed without composer.

Old scripts still work in Magento 2.3. To convert old scripts to new format, `Magento\Framework\Setup\Patch\PatchVersionInterface` should be implemented. This interface allows you to specify the setup version of the module in your database. If the version of the module is higher than or equal to the version specified in your patch, then the patch is skipped. If the version in the database is lower, then the patch installs.
