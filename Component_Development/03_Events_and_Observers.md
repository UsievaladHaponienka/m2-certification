**3. Events and Observers** (https://devdocs.magento.com/guides/v2.3/extension-dev-guide/events-and-observers.html)

Events are dispatched by modules when certain actions are triggred. Magento 2 also allows to create custom event.
Events are dispatched using `Magento\Framework\Event\ManagerInterface::dispatch` method. Interface should to be injected into class which should trigger some event:
```php
$this->eventManager->dispatch('my_module_event_after', ['myEventData' => $eventData]);
```
Second argument in optional. It contains array which can be used later in Observer.

**To trigger custom event you need to:**
- Declare this event with unique name in etc/events.xml.
- Inject Event Managet into your class and call it method 'dispatch' passing event name as argument.

Events.xml file can also be located under area folder - `etc/frontend` of `etc/adminhtml`. Observers associated with this event will watch for this events only when they are dispatched in corresponding area.

Observers are executed when associated event is dispatched. All observers are located in `<module-root>/Observer` directory and must implement  `Magento\Framework\Event\ObserverInterface`,
so all Observers must define `execute()` method. Observers have access to data passed while dispathing event:
```php
public function execute(\Magento\Framework\Event\Observer $observer)
{
	$myEventData = $observer->getData('myEventData');
}
```

Observers are "connected" to events in events.xml:
```xml
  <event name="my_module_event_before">
      <observer name="myObserverName" instance="MyCompany\MyModule\Observer\MyObserver" disabled="false" shared="true"/>
  </event>
```

Events.xml file from the whole magento are merged according to module load order. That means that it is possible to disable existing Observer by adding disabled="true" attribute:
```xml
    <event name="my_module_event_before">
        <observer name="myObserverName" disabled="true" />
    </event>
```
