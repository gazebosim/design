# Synchronization Between the GUI and Server ECM

## Goal
When creating, deleting, or modifying entities/components in the GUI _while paused_,
GUI changes should be reflected in the server once simulation is resumed.

## Approach
Update the ECM on the GUI side directly while simulation is paused (see [gazebo::GuiSystem::Update](https://github.com/ignitionrobotics/ign-gazebo/blob/ignition-gazebo6_6.0.0/include/ignition/gazebo/gui/GuiSystem.hh#L54)),
and then let the server ECM know what changes took place when simulation is resumed.
The server ECM should be updated to match the GUI ECM. Once the ECMs match, simulation will be carried out as normal.

The ECM can be updated via a [msgs::SerializedStateMap](https://github.com/ignitionrobotics/ign-msgs/blob/ignition-msgs8_8.0.0/proto/ignition/msgs/serialized_map.proto#L46).
So, the approach proposed here is to save each paused GUI action as a `msgs::SerializedStateMap`,
and then share all of these messages with the server when simulation is resumed.
The server will then call [gazebo::EntityComponentManager::SetState](https://github.com/ignitionrobotics/ign-gazebo/blob/ignition-gazebo6_6.0.0/include/ignition/gazebo/EntityComponentManager.hh#L638-L646) on every action message,
which will update the server’s ECM to match the changes made in the GUI’s ECM.

## Notes/Things to Consider

In order to share new, removed, and/or modified entities and components between the server and GUI, the components will need to be serializable.
There are some gazebo components that currently are not serializable.
We will need to add serializers to these components.

By modifying the GUI’s ECM directly whenever a user performs a GUI action while simulation is paused,
the component inspector will be updated appropriately since the component inspector uses the GUI’s ECM.

If each paused GUI action is saved and then shared with the server,
we now have a list of what actions took place (and in which order) in case we want to support undo/redo of these actions later on.

## Design

The design for this approach consists of adding a new GUI plugin called `GuiUserCommands`,
which would handle keeping track of all paused GUI actions.
Whenever a `gazebo::GuiSystem` performs an action that modifies the ECM,
the `GuiSystem` should create a `SerializedStateMap` msg representing the action, and share it with the `GuiUserCommands` plugin.
The `GuiUserCommands` plugin can interact with the `UserCommands` plugin on the server side to ensure that the server is updated according to paused GUI actions when simulation is resumed.

The orange boxes are GUI plugins, and everything above the dotted line represents behavior on the GUI side when simulation is paused. The blue boxes are server plugins, and everything below the dotted line represents behavior on the server side when simulation is no longer paused. The server behavior (below the dotted line) would only need to occur once whenever simulation transitions from paused -> playing.

![gui_server_ecm_design](images/GUI_Server_ECM_sync.png)
