## Tasks and Steps

A "task" represents one unit of work that can be assigned to a single Vehicle to complete.
A task can be decomposed into an ordered list of "steps", which are individual, atomic units of work that a vehicle can complete.

Each task is assigned a unique task ID upon creation. The ID is generated internally by the Dispatch API and should be
considered opaque.

### Task state transitions

A task may be in one of several states:

* `WAITING_FOR_ASSIGNMENT` ("Active")
* `DRIVING_TO_PICKUP` ("Active")
* `WAITING_FOR_PICKUP` ("Active")
* `DRIVING_TO_DROPOFF` ("Active")
* `COMPLETED`
* `CANCELLED`
* `REPLACED`

### Assignment

All tasks start out in the `WAITING_FOR_ASSIGNMENT` state. Once the task has been assigned to a vehicle, it moves into
the `DRIVING_TO_PICKUP` state.
Until the task's first step has been completed, it can still be unassigned from the vehicle (the task state reverts to `WAITING_FOR_ASSIGNMENT`), or reassigned to a different vehicle.
Once the task has any steps completed, the assignment of the task to the vehicle becomes permanent.
At this point, the task must be brought to termination -- either completed, cancelled, or replaced.

### Task definitions

Dispatch currently supports passenger tasks, which are tasks to pickup one group of passengers at a specific location,
and then drop them off at a different location. Passenger tasks contain three steps:

- Drive to pickup. This step is completed when the vehicle has arrived at the pickup location.
- Load resource. (Also called "waiting for pickup".) This step is completed once the passenger has entered the vehicle.
- Drive to dropoff. This step is completed when the vehicle has arrived at the dropoff location.

While tasks and task steps may change state (i.e. become completed or cancelled), the definition of a task cannot
change once the task is created. For example, a passenger task cannot change the location of its pickup or dropoff after
the task is created.
In order to support a passenger changing their pickup or dropoff location, we support a concept of task "replacement" -- see the "Replacement" section below.
A new task, with the changed locations, is created, and the old task is replaced by the new task (which has a new task ID).

### Termination

Tasks which are in the `COMPLETED`, `CANCELLED`, or `REPLACED` states are considered to be "terminated".
Once terminated, tasks cannot transition to any other state -- the termination states are final.
All other states are considered to be "active", where the task is either waiting to be assigned, or being actively performed.

A task being `COMPLETED` implies that all of the steps in the task have been completed.
Tasks may also be `CANCELLED`, which implies that either the passenger or the vehicle no longer wish to complete the rest of the task.
If a task is cancelled while the passenger is in the vehicle, the API will no longer track the passenger's state, and the vehicle is expected to resolve the situation on its own.
(For example, by pulling over and letting the passenger out.)

#### Replacement

A task may also be `REPLACED`, which indicates that both the passenger and the vehicle should switch to a new task ID.
(The new task ID can be found in the `replacement_task_id` field of PassengerTaskInfo.)
This is intended to provide a new task ID every time the "task definition" (e.g. the pickup and dropoff locations) changes, allowing clients to cache the task definition based on the task ID alone.

### Task creation constraints

A particular requestor (identified by `requestor_id`) may only have one active Task at a time. This models the
real-world constraint that no passenger may be picked up by more than one Vehicle at a time. The requestor may look up
their currently-owned active Task (if any) by calling `QueryCurrentTask`.

## Vehicles

The Dispatch API identifies the vehicles in your fleet by a user-specified `vehicle_id`. The `vehicle_id` is a string
that has only two restrictions: it must be unique (no two vehicles can have the same `vehicle_id`), and it must not be
empty.

### Vehicle state

The Dispatch API tracks the following aspects of each vehicle's state:

- Current location, represented as latitude/longitude coordinates (in WGS84). Use the `SetVehiclePosition` API to update
  the vehicle's position.
- Whether the vehicle is "ready for dispatch". Vehicles that are "ready for dispatch" may be assigned new passenger
  tasks. Use the `JoinFleet` or `SetReadiness` APIs to set a vehicle's "ready for dispatch" state.
- Whether the vehicle is "stale". Vehicles that haven't updated their position or requested their plan in 5 minutes are
  considered "stale" and won't receive new passenger tasks.

### Ready for dispatch

A vehicle that is "ready for dispatch", and not "stale", will be able to receive new trips in its plan.
If a vehicle becomes "not ready for dispatch" (i.e. the vehicle calls SetReadiness or JoinFleet with `ready_for_dispatch = false`), all of its currently-assigned tasks are immediately removed from the vehicle.
The `not_ready_action` controls what happens to those tasks -- whether they are re-assigned to other vehicles, or whether they are all cancelled.
If a vehicle becomes "stale" (i.e. a vehicle fails to contact the service for 5 minutes), no action is taken on the vehicle's currently-assigned tasks.

## Plans

Vehicles are assigned a vehicle "plan" by the Dispatch API. The plan is an ordered list of "waypoints", which are wrappers around task steps (see "Tasks and
Steps"). Plans will include all the (not-yet-completed) waypoints from all the tasks that the vehicle is assigned
to. The original order of the waypoints from a single task will be preserved (i.e. a dropoff cannot happen before a
pickup), but waypoints from different tasks may be interleaved.

The waypoints in a plan must be completed in order by the vehicle.

### Plan changes

While a vehicle is ready for dispatch, new tasks may be assigned to a vehicle, and their waypoints added to the vehicle's plan, at any time.
In addition, existing waypoints in a plan may be reordered to improve efficiency (for example, changing traffic conditions may make
it more efficient to perform the waypoints in a different order). Vehicles should periodically retrieve their current
plan from the Dispatch API to discover changes to their plan.

## Fleets

In the Dispatch API, both vehicles and tasks have an optional `fleet_id` property. The `fleet_id` specifies which
"Fleet" the given vehicle or task belongs to. Fleets are completely independent of each other. Vehicles can only perform
tasks that are in the same fleet as the vehicle. The optimization engine only considers one fleet at a time: tasks from
one fleet will not affect the optimization result of any other fleet.

Fleets are intended to help the user of the API logically separate different groups of vehicles and tasks, and make sure
that those groups do not intersect. For example, you may want run a development fleet of vehicles on a test track to
test the integration with the Dispatch API. Using a different `fleet_id` for the development fleet (versus the live
production fleet) ensures that test tasks will not be assigned to production vehicles and vice versa.

If the `fleet_id` is not specified, then the vehicle or task will belong to the default fleet (whose `fleet_id` is an
empty string).

### Setting Vehicle fleet

To set the `fleet_id` of a vehicle, call the `JoinFleet` API. Note that the `fleet_id` can only be changed while the
vehicle is not "ready for dispatch", and has no waypoints in its plan. A vehicle will never have waypoints in its plan
from a task with a different `fleet_id`.

Note: in the current Dispatch API, the `vehicle_id` of the vehicle must be unique across all fleets, no matter what the
vehicle's current `fleet_id` is.
