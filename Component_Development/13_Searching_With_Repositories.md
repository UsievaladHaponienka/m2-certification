# 12. Searching with repositories 
(https://devdocs.magento.com/guides/v2.3/extension-dev-guide/searching-with-repositories.html)

Repository must be stateless after instantiation. In means that that every method call should not rely on previous call and shoyul not affect further calls. If state is required for repository, *registry* pattern is required:

```php
// Magento\Customer\Model\ResourceModel\CustomerRepository::__construct() fragment
    public function __construct(
        //...
        CustomerRegistry $customerRegistry,
        //...
    ) {
    	//...
        $this->customerRegistry = $customerRegistry;
        //...
    }

    // Magento\Customer\Model\ResourceModel\CustomerRepository::getById()
    public function getById($customerId)
    {
        $customerModel = $this->customerRegistry->retrieve($customerId);
        return $customerModel->getDataModel();
    }
```

```php
// Magento\Customer\Model\CustomerRegistry::retrieve()
    public function retrieve($customerId)
    {
        if (isset($this->customerRegistryById[$customerId])) {	// Try lo find registred customer by Id
            return $this->customerRegistryById[$customerId];
        }
        /** @var Customer $customer */
        $customer = $this->customerFactory->create()->load($customerId);	// Get customer using customer factory model
        if (!$customer->getId()) {
            // customer does not exist
            throw NoSuchEntityException::singleField('customerId', $customerId);
        } else {
            $emailKey = $this->getEmailKey($customer->getEmail(), $customer->getWebsiteId()); // Register customer
            $this->customerRegistryById[$customerId] = $customer;
            $this->customerRegistryByEmail[$emailKey] = $customer;
            return $customer;
        }
    }
```

## SearchCriteria

Implementation of `Magento\Framework\Api\SearchCriteriaInterface` which allows to add conditions to request. Search criteria components:
- Filter. Instance of `Magento\Framework\Api\Filter`. Usage:
```php
$filter
    ->setField("url")
    ->setValue("%magento.com")
    ->setConditionType("like");	// ... WHERE `url` like %magento.com"
```

`Magento\Framwork\Api\FilterBuilder` is used to create filter and avoid shared instances.

- Filter Group. Instance of `Magento\Framework\Api\Search\FilterGroup`, allows to add filter collections to searchCriteria.
  - Filters are joined with `OR` condition inside Filter Group.
  - Filter Groups are joined with `AND` condition inside Search Criteria.

Usage:
```php
$filter1
    ->setField("url")
    ->setValue("%magento.com")
    ->setConditionType("like"); // ... WHERE `url` like "%magento.com"

$filter2
    ->setField("store_id")
    ->setValue("1")
    ->setConditionType("eq"); // ... WHERE `store_id` = 1

$filterGroup1->setFilters([$filter1, $filter2]); // ... WHERE (`url` like "%magento.com" OR `store_id` = 1)

$filter3
    ->setField("url_type")
    ->setValue(1)
    ->setConditionType("eq"); // ... WHERE `url_type` = 1 

$filterGroup2->setFilters([$filter3]);

$searchCriteria->setFilterGroups([$filterGroup1, $filterGroup2]); // ...WHERE (`url` like "%magento.com" OR `store_id` = 1) AND (`url_type` = 1)
```

`\Magento\Framework\Api\Search\FilterGroupBuilder` is used to create filter group and avoid shared instances.


- Sorting. Instance of `Magento\Framework\Api\SortOrder`. Usage:
```php
$sortOrder
    ->setField("email")
    ->setDirection("ASC");

$searchCriteria->setSortOrders([$sortOrder]);
```

`\Magento\Framework\Api\SortOrderBuilder` is used to create sort order and avoid shared instances.

- Pagination. SearchCriteria Methods `setPageSize()` and `setCurrentPage()` are used for pagination:
```php
$searchCriteria
    ->setPageSize(20)
    ->setCurrentPage(2); //show the 21st to 40th entity
```

### About `Builders`:
- `Builders` extend `\Magento\Framework\Api\AbstractSimpleObjectBuilder`
- `AbstractSimpleObjectBuilder::create()`:
```php
        $dataObjectType = $this->_getDataObjectType(); // Getting Instance class
        $dataObject = $this->objectFactory->create($dataObjectType, ['data' => $this->data]);
        $this->data = [];
        return $dataObject;
```
- ObjectFactory is `Magento\Framework\Api\ObjectFactory`. `ObjectFactory::create()`:
```php
    public function create($className, array $arguments)
    {
        return $this->objectManager->create($className, $arguments);
    }
```

So EntityBuilder class and ObjectFactory class are two levels of abstraction between ObjectManager and Entity.

## Search Result

Repository method `getList(SearchCriteria $searchCriteria)` should return `SearchResult`, implementation of `Magento\Framework\Api\SearchResultsInterface`. `Magento\Framework\Api\SearchCriteriaBuilder` is used:

```php
$filter = $this->filterBuilder
    ->setField(ProductInterface::NAME)
    ->setConditionType('like')
    ->setValue('%hoodie%')
    ->create();

$this->searchCriteriaBuilder->addFilters([$filter]);
$this->searchCriteriaBuilder->setPageSize(20);

$searchCriteria = $this->searchCriteriaBuilder->create();
$productsItems  = $this->productRepository->getList($searchCriteria)->getItems();
```
