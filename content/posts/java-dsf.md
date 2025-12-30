---
author: "Phong Nguyen"
title: "DSF"
date: "2025-05-13"
description: "Java: Program with DSF"
tags: ["java","eclipse"]   #tags search
FAcategories: ["syntax"]    #The category of the post, similar to tags but usually for broader classification.
FAseries: ["Themes Guide"]    #indicates that this post is part of a series of related posts
aliases: ["migrate-from-jekyl"]    #Alternative URLs or paths that can be used to access this post, useful for redirects from old posts or similar content.
ShowToc: true    # Determines whether to display the Table of Contents (TOC) for the post.
TocOpen: true    # Controls whether the TOC is expanded when the post is loaded. 
weight: 7    # The order in which the post appears in a list of posts. Lower numbers make the post appear earlier.
---
Explain how to use the DSF.
[Refer1](https://help.eclipse.org/latest/index.jsp?topic=%2Forg.eclipse.cdt.doc.isv%2Fguide%2Fdsf%2Fintro%2Fdsf_programming_intro.html)
<br>

![image](/images/dsf_overview.png)

## 1. Introduction
- Scenario: **Debugger View Fetching Data**
Context: We want to show variable values in the Variables View as the user steps through the code.

### 1.1. **Asynchronous**
  - Req: When the user steps through the program, the debugger needs to **fetch variable values** from the target (like a remote system). This operation might **take a few milliseconds or more**, depending on the connection. If we **block the UI thread, the UI freezes**.
  - Solution: Use an asynchronous method to fetch variable data

```java
void getVariableValue(String varName, DataRequestMonitor<String> rm) {
    // Imagine this takes time (e.g., contacting GDB or a remote debugger)
    // Simulate async call
    executor.execute(() -> {
        String value = remoteFetchValue(varName);  // Slow operation
        rm.setData(value);
        rm.done();
    });
}
```
  - Why Async?
    - Prevents blocking the UI.
    - Allows the Eclipse debug framework to continue updating other views.

### 1.2. **Synchronous** 
  - We want to implement a feature like **evaluate expression** that requires the value immediately. We don’t want to rewrite your entire logic using async callbacks just to get the result of getVariableValue().

  - Solution: Use a DSF Query to **wrap the async call in a synchronous interface**:

```java
String getVariableValueSync(String varName) {
    Query<String> query = new Query<>() {
        @Override
        protected void execute(DataRequestMonitor<String> rm) {
            getVariableValue(varName, rm);  // Wrap Asynchronous method
        }
    };

    executor.execute(query);

    try {
        return query.get();  // Blocks until result is ready (asynchronous method done  rm.done();)
    } catch (Exception e) {
        return "Error";
    }
}
```

  - Why Sync?
    - Some contexts expect a result immediately.
    - Simplifies logic when integrating with legacy synchronous systems.

### 1.3. Summary Table:

|Use Case | Method Type |	Reason |
|---------|-------------|----------|
|Fetching variable values in UI|	**Asynchronous**|	Avoid blocking UI thread|
|Nested UI event (e.g., button click)|	**Asynchronous**|	Keeps UI responsive|
|Performing batch operations|	**Asynchronous**|	Can parallelize or chain operations|
|Calling legacy blocking logic|	Synchronous|	Synchronous integration is easier|
|Evaluating expressions in scripts|	Synchronous|	Scripts expect immediate result|

## 2. Asynchronous Methods: Request Monitor , Data Request Monitor,  Multi-Request Monitor
- `Async methods is a way that use a callback object to indicate their completion`.

## 2.1. Request Monitor
- There is an standard **callback object** used in DSF, the request monitor, has following features:
  - **Executor**: executor to invoke the callback method
  - **Status**: indicating the success or failure of the callback method
  - **Callback methods**: methods which are invoked when the callback is invoked: handleCompleted(), handleOK(), handleError(),... -> overwrite these if need to perform additional process after async method completion.
  - **Parent request monitor**: 
  - Returning values to the caller. ( Data Request Monitor)
  - Done count. (Multi-Request Monitor)

## 2.2. Data Request Monitor
- The **Request Monitor** can return status of the async method but do not return a value to caller. **Data Request Monitor** can be used for that purpose.

## 2.3. Counting Request Monitor
- When we need to manage the completion of several request monitor. It is configured such that it's done() method needs to be called a count number of times before the callback method is invoked. 

## 2.4 Examples:
```java
import java.util.concurrent.Executor;

import org.eclipse.cdt.dsf.concurrent.DataRequestMonitor;
import org.eclipse.cdt.dsf.concurrent.ImmediateExecutor;
import org.eclipse.cdt.dsf.concurrent.RequestMonitor;

public class Test {

	public static void main(String[] args) {
		Executor executor = ImmediateExecutor.getInstance(); // Get the Executor
		RequestMonitor rm1 = new RequestMonitor(executor, null) {
			@Override
			protected void handleCompleted() {   // callback function 
				System.out.println("Fcn1 has done");
			}
		};
		asyncFcn1(rm1);

		DataRequestMonitor<Integer> rm2 = new DataRequestMonitor<>(executor, null) {
			@Override
			protected void handleCompleted() {  // callback function
				Integer data = getData();
				System.out.println("Fcn2 has done");
				System.out.println("Returned data: " + data);
			}
		};
		asyncFcn2(2, rm2);
	}

	static void asyncFcn1(RequestMonitor rm) {
		System.out.println("Run Fcn1");
		rm.done(); // notify that the async method done -> will run the callback function 
	}

	static void asyncFcn2(int value, DataRequestMonitor<Integer> rm) {
		System.out.println("Run Fcn2");
		rm.setData(value);
		rm.done();
	}
}
```

- Output:
```powershell
Run Fcn1
Fcn1 has done
Run Fcn2
Fcn2 has done
Returned data: 2
```

## 3. Concurrency
### 3.1. Query ( implements Runnable)
- Using a Query can use a execute() (overwrite) implementation in order **to call other asynchronous method** from a **sync method**
```java

import java.util.concurrent.Executor;

import org.eclipse.cdt.dsf.concurrent.DataRequestMonitor;
import org.eclipse.cdt.dsf.concurrent.ImmediateExecutor;
import org.eclipse.cdt.dsf.concurrent.Query;

public class TestQuery {

	public static void main(String[] args) {
		Executor executor = ImmediateExecutor.getInstance();
		int result = syncAdd(2, 3, executor);
		System.out.println("Result: " + result); // Expected output: Result: 5
	}

	// Asynchronous method ( fetch value etc, take more time here)
	static void asyncAdd(int a, int b, DataRequestMonitor<Integer> rm) {
		rm.setData(a + b);
		rm.done();
	}

	// Synchronous method using DSF Query to call a async method
	static int syncAdd(final int a, final int b, final Executor executor) {
		Query<Integer> query = new Query<>() {
			@Override
			protected void execute(final DataRequestMonitor<Integer> rm) {
				asyncAdd(a, b, rm); // Call async method inside query
			}
		};

		executor.execute(query); // Submit the query to the executor

		try {
			return query.get(); // Blocks and waits for result
		} catch (Exception e) {
			e.printStackTrace();
			return -1; // Return error code on failure
		}
	}

}
```

### 3.2. Synchronization
- DSF uses a single-threaded executor (DSF Executor thread) as the primary mechanism for safe access to date. (race conditions + deadlocks)
 
### 3.3. Annotations
- DSF defines a number of annotations that can be used to determine what are the rules governing access to the various data objects.

## 4. Services
### 4.1. OSGi
- DSF builds on top of OSGi service, so there are some of the OSGi service APIs basic:
  - Registration: BundleContext.registerService(), ServiceRegistration.unregister() 
  - References: BundleContext.getServiceReference(s) .getAllServiceReferences()
  - Events: ServiceListener

### 4.2. DSF Session
- DSF services are organized into sessions to help coordinate and distinguish the services. A DSF session is a way to associate a set of DSF services
- DSF Session features include:
  - Session ID
  - DSF Executor: Each session has a DSF Executor. All the services registered with the same session share a single DSF Executor. 
  - Service Events
  - Model Adapters
(Một session đại diện cho một quá trình đang hoạt động. Mỗi session gắn với một executor, và chứa các services cần thiết (breakpoint, memory, register…).)

### 4.3. DSF Executor
For DSF, the main rule for executors is that they have to use a single thread to execute the runnable and that the runnables be executed in the order that they were submitted
```java
void java.util.concurrent.Executor.execute(
Runnable command
)
```
(Nó là một thread executor (kiểu như một thread riêng biệt) mà DSF sử dụng để chạy tất cả các tác vụ (services, events) một cách tuần tự. Mục tiêu là đảm bảo mọi thứ chạy trên một thread riêng biệt để tránh xung đột.)

### 4.4. DSF Services Tracker (Tracker)
- Both `org.osgi.util.tracker.ServiceTracker` and `org.eclipse.cdt.dsf.service.DsfServicesTracker` are used to **track and retrieve services** dynamically.
- DsfServicesTracker:
  - Tracks all services within a given DSF session
  - getService(s)
  - <constructor>: DSF service trackers can be used after being constructor
  - dispose(): disposes and un-gets all service references

(Là công cụ để tra cứu các service cụ thể bên trong một session.
Ví dụ bạn có thể dùng nó để lấy IBreakpointService, IMemoryService, IStackService, v.v.)

### 4.5. Initialization/ Shutdown
- Every DSF service must implement the initialize() and shutdowns() methods.

### 4.6. Events
- To generate an event, service must call : `DsfSession.dispatchEvent(Object event, Dictionary<?, ?> serviceProperties)`
- To receive DSF events, client must: 
  - Declare a public event listener method, which takes an event parameter.
  - Add itself as a service listener by calling `DsfSession.addServiceEventListener()`


### 4.7. Example
```java
    DsfExecutor fExecutor = new DefaultDsfExecutor();    // dsf executor use for a new session with some services
	DsfSession fSession = DsfSession.startSession(fExecutor, "Timers(DSF Example)"); // dsf session
	DsfServicesTracker fServices = new DsfServicesTracker(DsfExamplesPlugin.getBundleContext(), fSession.getId()); // dsf services tracker for this session

    // Start some services    
	ServicesStartupSequence startupSeq = new ServicesStartupSequence(fSession);
	fSession.getExecutor().execute(startupSeq);
	try {
		startupSeq.get();    // blocks until startup done
	} catch (InterruptedException e) {
		assert false;
	} catch (ExecutionException e) {
		assert false;
	}

    // Get and execute a service (e.g. TimerService class)
	fExecutor.execute(() -> fServices.getService(TimerService.class).startTimer());
```

## 5. Data Model 
- Introduce the natural structure of the data that is being retrieved by the DSF services.
-  [target → process → thread → stack frame → variable, v.v.]
### 5.1. IDMContext
- Represents a handle to a chunk data in Data Model
- Features:
  - hierarchical: Contexts can have other contexts as parents.
  - extends IAdaptable: allows decorators, retargetable actions,..
  - associated with a DSF Session: IDMContext.getSessionID()

### 5.2. DMContext
- The utility class contains a few static methods:
  - `getAncestorOfType()`:  Searches for a context of a specific type in the hierarchy of the given context.
  - `isAncestorOf()`: checks whether the one contexts is in the hierarchy of the other
  - `toList()`

### 5.3. Context Hierarchy
- We can  get the immediate ancestors of a given context and following the parents' parents allows clients to traverse the full hierarchy of a context.
- E.g.
```java
@Immutable
	public static class AlarmDMContext extends AbstractDMContext {
        //
	}

@Immutable
	public static class TimerDMContext extends AbstractDMContext {
        //
	}

    
public boolean isAlarmTriggered(AlarmDMContext alarmCtx){
    		TimerDMContext timerCtx = DMContexts.getAncestorOfType(alarmCtx,
				TimerDMContext.class);

            if(timerCtx != null){
                // Returns the ancestor if found
            }
}
```



## 0. Timer Clone From CDT/DSF Example

1. Contribute to Shop view
```xml
<plugin>
   <extension
         point="org.eclipse.ui.views">
      <category
            id="cdt-dsf-timers-clone.category.id"
            name="My Category Views">    # category views
      </category>
      <view
            category="cdt-dsf-timers-clone.category.id"
            class="views.TimersView"    # the class extend ViewPart
            icon="icons/timer.gif"
            id="cdt-dsf-timers-clone.view.id"
            name="Timers View Clone" # view
            restorable="true">
      </view>
   </extension>

</plugin>
```

- Use `org.eclipse.ui.views` extension to contribute the category views/view

- Bundle-Activator is the fully qualified name of the BundleActivator implementation that will be used to start and stop the bundle, and it refers to the class we’ve just written. (`Bundle-Activator: activator.TimerPluginActivator`)

2. Create the View
3. Design

**- org.eclipse.cdt.dsf.service.IDsfService**
@ConfinedToDsfExecutor(value="getExecutor")
The interface that all DSF services must implement. It only provides a few features to help manage and identify the services using the OSGI services framework.
Each service should register itself with OSGI services framework using the BundleContext.registerService() method. And each service should use the session ID that it is registering with as one of the service properties. If there is more than one instance of the service to be instantiated for a given session, additional properties should be used when registering the service to allow clients to uniquely identify the services.
By convention, all methods of DSF services can be called only on the dispatch thread of the DSF executor that is associated with the service. If a service exposes a method that is to be called on non-dispatch thread, it should be documented so.

**- org.eclipse.cdt.dsf.service.DsfSession**
@ConfinedToDsfExecutor(value="getExecutor")
Class to manage DSF sessions. A DSF session is a way to associate a set of DSF services that are running simultaneously and are interacting with each other to provide a complete set of functionality.
Properties of a session are following:
1. Each session is associated with a single DSF executor, although there could be multiple sessions using the same executor.
2. Each session has a unique String identifier, which has to be used by the services belonging to this session when registering with OSGI services.
3. Each session has its set of service event listeners.
4. Start and end of each session is announced by events, which are always sent on that session's executor dispatch thread.

- **org.eclipse.cdt.dsf.concurrent.DsfExecutor**
@ThreadSafe
DSF executor service. Implementations of this executor must ensure that all runnables and callables are executed in the same thread: the executor's single dispatch thread.
Note: A DSF executor dispatch thread does not necessarily have to be exclusive to the executor, it could be shared with another event dispatch service, such as the SWT display dispatch thread.

**- org.osgi.framework.ServiceRegistration<S>**
@ProviderType
A registered service.
The Framework returns a ServiceRegistration object when a BundleContext.registerService method invocation is successful. The ServiceRegistration object is for the private use of the registering bundle and should not be shared with other bundles.
The ServiceRegistration object may be used to update the properties of the service or to unregister the service.

**- org.eclipse.cdt.dsf.service.DsfServicesTracker**
@ConfinedToDsfExecutor(value="DsfSession.getSession(sessionId).getExecutor()")
Convenience class to help track DSF services that a given client needs to use. This class is similar to the standard OSGI org.osgi.util.tracker.ServiceTracker class, with a few differences:
1. This class is assumed to be accessed by a single thread hence it has no synchronization built in, while OSGI ServiceTracker synchronized access to its data.
2. This class is primarily designed to track multiple services of different type (class), while OSGI ServiceTracker is designed to work with single class type, with optional filtering options.
3. This class uses knowledge of DSF sessions to help narrow down service references.
4. OSGI Service tracker explicitly listens to OSGI service startup/shutdown events and it will clear a reference to a service as soon as it's shut down. Since version 2.0, this class listens to service unregister events as an indication of service shutdown. In the case of an unregister event, this class will clear the reference to that service.

That said, it might be more convenient for certain types of clients to use OSGI Service tracker for the additional features it provides.
The ServiceTracker class simplifies using services from the Framework's service registry.

A **ServiceTracker** object is constructed with search criteria and a ServiceTrackerCustomizer object. A ServiceTracker can use a ServiceTrackerCustomizer to customize the service objects to be tracked. The ServiceTracker can then be opened to begin tracking all services in the Framework's service registry that match the specified search criteria. The ServiceTracker correctly handles all of the details of listening to ServiceEvents and getting and ungetting services.
The getServiceReferences method can be called to get references to the services being tracked. The getService and getServices methods can be called to get the service objects for the tracked service.
The ServiceTracker class is thread-safe. It does not call a ServiceTrackerCustomizer while holding any locks. ServiceTrackerCustomizer implementations must also be thread-safe.

Khái niệm chính:
Bundle (Plugin): Một đơn vị triển khai độc lập, có thể được cài đặt, khởi động, dừng và gỡ bỏ trong môi trường OSGi, mỗi bundle có một BundleContext riêng.
BundleContext: Cung cấp API cho bundle tương tác với OSGi Runtime (Framework), bao gồm việc lấy cấu hình, đăng ký/lấy dịch vụ, quản lý sự kiện.
OSGi Service: Các đối tượng Java được đăng ký vào Service Registry và có thể được các bundle khác tìm thấy và sử dụng, tạo thành một "service-oriented architecture" (kiến trúc hướng dịch vụ). 


ImmediateInDsfExecutor

Executes immediately in the current thread.

No thread switching, no queue.

Synchronous behavior.

Mainly used for testing, bootstrap, or when you are already on the DSF executor thread.

Dangerous if misused → can break DSF threading rules or cause re-entrancy issues.

In short:

“Run this code right now, on the same thread.”

DefaultDsfExecutor

Uses a dedicated DSF executor thread.

Tasks are queued and executed sequentially.

Asynchronous and thread-safe.

This is the recommended and standard executor for DSF services.

Prevents race conditions and UI freezes.

In short:

“Schedule this code to run later on the DSF thread.”


VM Element	Design Patterns
VM Adapter	Facade, Strategy, Mediator
VM Provider	Controller, Chain of Responsibility
VM Node	Composite, Factory, Template Method
VM Context	Adapter, Value Object, Proxy
Whole system	MVC/MVVM, Observer, Command, Async


> DMContext là một “handle ổn định” đại diện cho một đối tượng trong DSF Data Model, dùng để định danh và truy vấn dữ liệu một cách an toàn trong kiến trúc async.