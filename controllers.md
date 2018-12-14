## Controllers

To reduce complexity, all controllers are packaged and shipped in a single daemon named kube-controller-manager. 

### Controller Components
There are two main components of a controller: 
1. Informer/SharedInformer
2. Workqueue
##### Informer 
In order to retrieve an object's information, the controller sends a request to Kubernetes API server.

The client-go library provides the Listwatcher interface that performs an initial list and starts a
watch on a particular resource:
  ````
  lw := cache.NewListWatchFromClient(
        client,
        &v1.Pod{},
        api.NamespaceAll,
        fieldSelector)
  ````
All of these things are consumed in Informer. A general structure of an Informer is described below:
  ````
  store, controller := cache.NewInformer {
  	&cache.ListWatch{},
  	&v1.Pod{},
  	resyncPeriod,
  	cache.ResourceEventHandlerFuncs{},
 ````
Infromer is has been succeded by SharedInformer.
###### ListWatcher
Listwatcher is a combination of a list function and a watch function for a specific resource in a specific
namespace. 

This helps the controller focus only on the particular resource that it wants to look at. The 
field selector is a type of filter which narrows down the result of searching a resource like the controller 
wants to retrieve resource matching a specific field.
  ```$xslt
cache.ListWatch {
	listFunc := func(options metav1.ListOptions) (runtime.Object, error) {
		return client.Get().
			...
			.Do().Get()
	}
	watchFunc := func(options metav1.ListOptions) (watch.Interface, error) {
		options.Watch = true
		return client.Get().
			Namespace(namespace).
			...
			.Watch()
	}
}
```
  
 ###### Resource Event Handler
 Resource Event Handler is where the controller handles notifications for changes on a particular resource:
 ```$xslt
type ResourceEventHandlerFuncs struct {
	AddFunc    func(obj interface{})
	UpdateFunc func(oldObj, newObj interface{})
	DeleteFunc func(obj interface{})
}type ResourceEventHandlerFuncs struct {
 	AddFunc    func(obj interface{})
 	UpdateFunc func(oldObj, newObj interface{})
 	DeleteFunc func(obj interface{})
 }
```
##### ResyncPeriod
ResyncPeriod defines how often the controller goes through all items remaining in the cache and fires the UpdateFunc again.

It's extremely useful in the case where the controller may have missed updates or prior actions failed. However, 
if you build a custom controller, you must be careful with the CPU load if the period time is too short.

#### SharedInformer
The SharedInformer helps to create a single shared cache among controllers. 

This means cached resources won't be duplicated and by doing that, the memory 
overhead of the system is reduced. Besides, each SharedInformer only creates a
single watch on the upstream server, regardless of how many downstream 
consumers are reading events from the informer. This also reduces the load on 
the upstream server. This is common for the kube-controller-manager which has
so many internal controllers.
```$xslt
lw := cache.NewListWatchFromClient(…)
sharedInformer := cache.NewSharedInformer(lw, &api.Pod{}, resyncPeriod)
```

##### Workqueue
The SharedInformer can't track what each controller is up to (because it's shared)
, so the controller must provide its own queuing and retrying mechanism (if required).
Hence, most Resource Event Handlers simply place items onto a per-consumer workqueue.
````
controller.informer = cache.NewSharedInformer(...)
controller.queue = workqueue.NewRateLimitingQueue(workqueue.DefaultControllerRateLimiter())

controller.informer.Run(stopCh)

if !cache.WaitForCacheSync(stopCh, controller.HasSynched)
{
	log.Errorf("Timed out waiting for caches to sync"))
}

// Now start processing
controller.runWorker()
````