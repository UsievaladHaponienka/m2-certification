**Magento 2 database transactions**
(https://webkul.com/blog/database-transactions-magento2/)

Transaction is a logical unit that is executed independently for data retrieval or updates. In relational DB transactions must follow **ACID** principle: 
- *A (Atomicity).* Transaction must be either fully completed and saved (committed) or completely undone (rolled back). "All or nothing".
- *C (Consistency).* Transaction must be fully compliant with database. For example, transaction must not be committed if it tries to enter letter in column, which has numeric type.
- *I (Isolation).* Transaction data must not be available to other transactions until the original transaction is committed or rolled back.
- *D (Durability).* Transaction data changes must be available, even in the event of database failure.

Each transaction must end with either commit (all changes are written to DB) or rollback (all changes are reverted).

Magento 2 example transaction flow (saving product entity):

```
01. Product model -> save().
02. --Product resource -> save().
03. ----Begin transaction (0 level).
04. ------Before product save events.
05. ------Creating new product or updating existing one.
06. ------After product save events.
07. --------One of ofter product save events is saving of another entity - CatalogInventory Stock -> save().
08. ----------Catalog inventory stock resource -> save().
09. ------------Begin transaction (1 level).
10. --------------Before stock save events.
11. --------------Creating/Updating stock entity.
12. --------------After product save events.
13. ------------Commit of **1st level**. Callbacks are not executed.
14. ----Commit of **0 level**.
15. Callbacks executed from highest level to lowest.
```

**Consequences:**
1. All changes to database are actually applied after 0 level commit. Thus if any error occurs during saving process, all changes are reverted:
```php
<?php
$entityOne->save(); // Triggers save of entityTwo and entityThree 
$entityTwo->save(); // Success
$entityThree->save(); // Error, for example, duplicated unique key
```

Though entityTwo "was saved", **ALL** changes are reverted (according to *Atomicity* principle).

2. Callbacks are executed *after* 0 level commit. So if there are errors during callbacks, nothing is reverted, only logged (transaction was already finished with commit result).
