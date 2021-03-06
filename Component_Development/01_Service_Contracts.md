**1. Service Contracts** (https://devdocs.magento.com/guides/v2.3/extension-dev-guide/service-contracts/design-patterns.html)

Secvice contract is a set of PHP interfaces which allows to inject and use intefaces instead of classes.
Service contract must define Data Interfaces and Service Interfaces. Also Search Result Interface should be defined to use it in repository method getList();

**Service Interfaces:**
- Repository Interfaces
- Management Interfaces
- Metadata Interfaces

**Repository Interfaces provide access to data entities. They must define following methods:**
- save (data entity interface) - If an ID is not specified, creates a record. If an ID is specified, updates the record for the specified ID.
- get(id) - Performs a database lookup by ID.Returns a data entity interface, such as CustomerInterface or AddressInterface.
- getList(SearchCriteria) - Performs a search for all data entities that match specified search criteria. Returns a search results interface that gives access to the set of matches.
- delete(data entity interface) - Deletes a specified entity. The entity contains the key (ID).
- deleteById(id) - Deletes a specified entity by key (ID).

**Management Interfaces provide functions which are not related with repositories, for example:**
- createAccount()
- changePassword()
- activate()
- validate(), etc.
