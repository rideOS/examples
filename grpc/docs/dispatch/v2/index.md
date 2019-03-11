rideOS Dispatch API v2
======================

The rideOS dispatch API provides the ability to:

1. Maintain the state of a fleet of vehicles serving a variety of tasks.
2. Automatically assign tasks to vehicles to maximize efficiency.

The dispatch v2 API is part of rideOS' Fleet Control series of products.

## Basic call flows

### Vehicle flowchart

<img src="flowchart_vehicle.png" />

Initially, a vehicle starts in the "not ready for dispatch" state. Call `SetReadiness` to make the vehicle "ready for
dispatch" (also known as "online"). (If using fleets, call `JoinFleet` instead.) Once a vehicle is ready for dispatch,
the backend will start to assign tasks to that vehicle automatically. Poll on the `SyncVehicleState` API regularly while
the vehicle is ready for dispatch -- this informs the backend of the vehicle's current position, and informs the caller
(i.e. the vehicle) of any changes to its Plan. Once the vehicle wishes to stop receiving new tasks, call the
`SetReadiness` API again to make the vehicle "not ready for dispatch".

### Task flowchart

A request to pick up a passenger and drop them somewhere else, or a request to pick up a package and deliver it, or a
request to make sure that a vehicle is available at a certain spot before some anticipated passenger demand (before a
big event ends); these are all considered Tasks to the dispatch backend.

<img src="flowchart_task.png" />

To request a passenger task (a task to pickup and dropoff a group of passengers), call `RequestPassengerTask`. While the
task is ongoing, poll on `GetPassengerState` regularly to check on the status of the task. The returned
`PassengerTaskInfo` object says which vehicle has been assigned to take the task (if any), and how far along the task
has progressed (i.e. if the passenger has been picked up yet, etc.).

Note that, upon startup of the passenger's app, the passenger may already have requested a task. We recommend calling
`GetPassengerState` as soon as possible after the app starts, to make sure that the passenger's on-trip status is
reflected in the app state. The API does not currently allow requesting more than one passenger task at a time.

## Data model

More details about the representation and assumptions made about vehicles and tasks can be found in the [data model 
guide](data_model_guide.md).
