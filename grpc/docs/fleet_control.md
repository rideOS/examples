Fleet Control
=============

Fleet Control is an end-to-end solution for performing ride-hailing and other tasks across a fleet of vehicles.

Components
----------

Fleet Control consists of the following components:

### APIs

Fleet Control currently offers two APIs:

* `dispatch.v2` -- the `dispatch.v2` API provides core state management functionality.
  Access invididual tasks and vehicles, or make queries across an entire fleet.
  Can be called through a backend-to-backend interaction, or directly by mobile apps.
* `ride_hail.v1beta` -- the `ride_hail.v1beta` API provides convenince functionality for mobile apps that offer ride-hailing functionality.

### Dashboard

We also provide a web-based dashboard which integrates seamlessly with Fleet Control.
All fleets, vehicles, and tasks that are managed using Fleet Control can be viewed and edited using the dashboard.
The dashboard also offers a simulation tool, which can be used to experiment with Fleet Control functionality without setting up a real fleet of vehicles.
Open the dashboard by going to [app.rideos.ai](https://app.rideos.ai).

### Mobile apps

We offer two mobile apps (one for passengers, and one for drivers) that use the Fleet Control APIs to provide an end-to-end ride-hailing experience.
These apps can be rebranded and used directly as white-label apps, or you can build an entirely custom app using these apps as an SDK.
For more information about the mobile apps, please contact our team directly at contact@rideos.ai.

Further documentation
---------------------

More detailed documentation can be found for each API:

* [dispatch.v2 documentation](dispatch/v2/index.md)

Fleet Control vs Fleet Planner
------------------------------

rideOS offers two dispatch-related products at different abstraction levels: Fleet Control and Fleet Planner.

Fleet Control is a state management system that tracks the status of individual tasks and vehicles, and the assignments from tasks to vehicles.
A ride-hailing mobile app needs to call APIs that answer "what is the status of my trip?" or "where is the vehicle that is assigned to pick me up?"
In order to answer these queries, there must be a system in place to persist and keep track of the state associated with each individual vehicle and task.
Fleet Control is rideOS' product for task and vehicle state management.

Fleet Planner is a lower-level API that we offer to help your fleet make better dispatch decisions.
The type of query that Fleet Planner answers is of the form "given these vehicles and these tasks, which task(s) should each vehicle take?".
Fleet Control uses Fleet Planner in the background as its optimization method.
Fleet Planner does not offer any functionality to query the state of an individual task or vehicle -- use Fleet Control instead to get that functionality.
