# Synchronization Between the GUI and Server ECM

## Goal
When creating, deleting, or modifying entities/components in the GUI _while paused_,
GUI changes should be reflected in the server once simulation is resumed, or stepped while paused.

## Approach
If a user performs actions that modify/create/remove entities while simulation is paused, the ECM on the GUI side is updated directly to reflect these actions.
The server's ECM does not reflect these user actions yet, but will be notified of the changes that took place once simulation is resumed or stepped while paused.
When the server ECM is notified of these changes, it will be updated to have any changes made to the GUI ECM (unless the GUI and server are run in the same process, which will be discussed below).
Once the server ECM is updated, simulation will be carried out as normal.

The image below depicts how the GUI and server interact, including how changes to the GUI's ECM can be propagated to the server's ECM.
The GUI and server interaction can be initiated by an event on the GUI side, which will be processed first and then connect to the server through a service call (green arrows).
The GUI and server interaction can also occur directly through a service call, skipping the GUI event (red arrows).
GUI ECM changes are only propagated to the server if the GUI and server interaction is triggered by an event, because the event will save the GUI's ECM state before making a service call that is received by the server.
If GUI and server interaction is done directly through a service, then there's no way to propagate GUI ECM changes to the server.
The black arrows in the image are things that occur in both the event and service approach.

![gui_server_ecm_design](images/gui_server_ECM_sync_design.png)

## Notes/Things to Consider

### How is the ECM shared and processed between the GUI and server?
The GUI will call [EntityComponentManager::State](https://ignitionrobotics.org/api/gazebo/6.0/classignition_1_1gazebo_1_1EntityComponentManager.html#a8dbc9cf1c9eb4af335aebc178b6cb6f7) to serialize its ECM into a `msgs::SerializedState`, and then share this serialized ECM with the server.
Once the server receives this `msgs::SerializedState`, the server will call [EntityComponentManager::SetState](https://ignitionrobotics.org/api/gazebo/6.0/classignition_1_1gazebo_1_1EntityComponentManager.html#a573b9551891a135bce602344e73a2a36) to apply changes in the GUI's ECM to the server's ECM.

### Server/Client Same Process
When running the server and client in the same process, the server and GUI share the same ECM.
In this scenario, the GUI (`gazebo::GuiRunner`) does not need to share its ECM with the server (`gazebo::SimulationRunner`), but the server still needs to make sure that server plugins are updated according to any changes made to the ECM while paused.

### Updating server plugins
While calling `EntityComponentManager::SetState` will help ensure that server ECM is updated according to changes in the GUI's ECM, system plugins will also need to be updated based on the changes
that took place while simulation was paused.
Here are a few scenarios that will need to be considered:
* Is the physics system ready to handle new links being added to a model?
* Does the sensors system correctly handle a geometry change on a visual?
* What happens to the diff-drive plugin if a joint suddenly disappears?

While the `ign-gazebo` API gives plugins a way to check what entities are new or removed, it does not provide a way to check what entities have recently changed components.
To get around this, when GUI plugins update an entity in the ECM, a "Recreate" component should be attached to the entity that flags it as an entity that was modified while simulation was paused.
When the `gazebo::SimulationRunner` processes changes made to the GUI's ECM, it should also search for entities with the "Recreate" component, and then delete/re-create these entities.
That way, server plugins can listen for newly created entities and operate on them accordingly.

### Undo/Redo
The implementation depicted above does not support undo/redo.
Undo/redo will be difficult to support while simulation is running because of things like undoing/redoing physics, but perhaps it can be supported to some extent by bookmarking ECM states at various times.
`msgs::SerializedState` may be useful for ECM bookmarking.

### Component Serialization
In order to share new, removed, and/or modified entities and components between the server and GUI, the components will need to be serializable.
There are some gazebo components that currently are not serializable.
We should not rely on components that can't be trivially serialized, such as `components::ModelSdf`.

### Component Inspector (GUI)
By modifying the GUI’s ECM directly whenever a user performs a GUI action while simulation is paused,
the component inspector will be updated appropriately since the component inspector uses the GUI’s ECM.
