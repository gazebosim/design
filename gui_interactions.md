# GUI interactions

This document is meant to inform and guide the development of various GUI
interactions for Ignition Gazebo.

## Overview

Ignition Gazebo's graphical interface's high-level goals are to:

1. **Introspect** a running or recorded simulation
2. **Interact** with a running or recorded simulation
3. **Edit** simulation worlds and robots

**Introspection** allows users to see how objects are positioned through a 3D
view, the precise position of objects through text widgets, how velocities
are changing over time through plots, etc.

**Interaction** allows users to apply forces to a robot while it moves,
to teleport objects in the scene while simulation is running, etc.

**Editing** allows users to arrange world environments, assemble new
robots and objects, modify existing objects, etc.

The main difference between interaction and editing is that the former must
happen while the simulation is running, while the latter doesn't. But in several
aspects, their functionalities overlap. For example, teleporting an object may
be useful while the simulation is running to test whether a robot will keep
following it. And teleporting objects is also essential to creating a new world
environment.

It could be argued that every aspect of editing could be helpful for
interaction. Even actions like changing a link's mass or adding a new link while
physics is running will be helpful to some user. However, it's important to keep
in mind that handling interactions is considerably more complicated than
handling editing, primarily because physics engines are usually designed to deal
with bodies that have immutable kinematics and dynamics.

With that in mind, this design document proposes the separation of features into
interaction and editing. The central difference between them being that editing
features can't happen while physics is running, while interaction features can
happen any time. In other words, we will have two separate modes of operation:

* **Edit mode**: whenever simulation is **paused**
* **Interactive mode**: whenever simulation is **playing**

Different modes of operation are common among software which allows both
creating and running content. Think about how slides can't be edited while
they're being presented, or how video editors will show short previews while
editing a video, but once the video is generated it can no longer be edited.
Closer to the robotics world, we have game engines which often have a clearly
separate run step for the game to be tested.

While implementing these different modes of operation, it's important to:

* Keep the user interface consistent across modes.
* Reduce context switching when changing modes.
* Reduce code duplication for different modes.

To keep the UI's look and feel consistent across modes, the same GUI plugins
should be used for interaction and editing whenever possible. These plugins
should be aware of the current simulation paused state, and enable features
as needed. For example, all fields in the Component Inspector should be writable
while simulation is paused, but only a few fields, such as model poses, should
be writable while simulation is running.

## Mode switching

* Switching between Edit and Interactive modes will be accomplished through the
play / pause button.
* It's up to each GUI plugin to be aware of the current simulation mode and
behave accordingly.
* SDF generation and saving won't be performed when exiting edit mode. Instead,
saving can be performed at any time from any mode based on the current state.
* All entities are editable while in edit mode.

## Lessons from Gazebo-classic

* Exiting the model editor is time consuming because we always save a new SDF
model. If we'll be using the play button as a toggle for edit mode, we can't do
the same so we don't hang every time the user unpauses.

* The different interfaces for visualizing properties during simulation and
during editing caused confusion for users.

* The representation of a model in memory was different according to the
simulation mode. This meant double the code to be maintained.

## Features

### A. User camera controls

#### A.1.Mouse orbiting

* **Modes**: Edit / Interactive
* **Priority**: Must have
* **References**
    * [Ignition Citadel: Understanding the GUI](https://ignitionrobotics.org/docs/citadel/gui)
* **User stories**
    * User wants to look at different parts of the world.
* **Status**: ✅ Available since Acropolis.

#### A.2. Preset view angles

* **Modes**: Edit / Interactive
* **Priority**: Should have
* **References**
    * [Ignition Citadel: View Angle](https://ignitionrobotics.org/docs/citadel/manipulating_models#view-angle)
* **User stories**
    * User is constructing a world and wants to check how models are aligned from
      different angles.
* **Status**: ✅ Available since Blueprint.

#### A.3. Move to entity

* **Modes**: Edit / Interactive
* **Priority**: Should have
* **References**
    * [Ignition Citadel: The Scene](https://ignitionrobotics.org/docs/citadel/gui#the-scene)
* **User stories**
    * User is inspecting a large world, and they want to find a specific
      entity.
* **Status**: ✅ Available since Blueprint.

#### A.4. Follow entity

* **Modes**: Interactive
* **Priority**: Should have
* **References**
    * [Ignition Citadel: The Scene](https://ignitionrobotics.org/docs/citadel/gui#the-scene)
* **User stories**
    * User is simulating a mobile robot and wants to keep it in view at all
      times.
* **Status**: ✅ Available since Blueprint.

### B. Geometry creation

Features related to creating and importing geometric shapes to be used
as rigid bodies in simulation.

#### B.1. Import 3D meshes

* **Modes**: Edit
* **Priority**: Must have
* **References**
    * [Gazebo tutorial](http://gazebosim.org/tutorials?tut=model_editor#Addmeshes)
        * Gazebo-classic's default behavior is to create a link and use the
          mesh as geometry for both visual and collision.
* **User stories**
    * User has a 3D mesh and wants to create a rigid model from it.
* **Status**: ⌛

Import a mesh file from the local file system to use as a visual or collision
geometry.

Ignition supports DAE, OBJ and STL meshes. Implementation of the importing
widget would involve opening a file browser and spawning the selected file into
simulation.

#### B.2. Extrude 2D shapes

* **Modes**: Edit
* **Priority**: Could have
* **References**
    * [Gazebo tutorial](http://gazebosim.org/tutorials?tut=extrude_svg)
* **User stories**
    * User has a 2D image and wants to extrude that as a geometry for a model.
* **Status**: ⌛

Extrude a 2D file from the local file system into a 3D shape to use as a visual
or collision geometry.

Like on Gazebo-classic, the widget could load an SVG file, let the user choose
a few parameters, and spawn the shape as collision and visual. Polyline support
will also need to be added.

#### B.3. Insert simple shapes

* **Modes**: Edit / Interactive
* **Priority**: Must have
* **References**
    * [Gazebo tutorial](http://gazebosim.org/tutorials?tut=model_editor#Addsimpleshapes)
* **User stories**
    * User wants to create a simple vehicle composed of boxes and cylinders.
* **Status**: ✅ Available since Blueprint for inserting as whole models.

Choose a primitive shape to use as a visual or collision geometry with default
properties. Support box, cylinder and sphere.

#### B.4. Insert lights

* **Modes**: Edit / Interactive
* **Priority**: Must have
* **References**
    * [Issue: Insert lights from the GUI](https://github.com/ignitionrobotics/ign-gazebo/issues/119)
* **User stories**
    * User needs to light up a specific part of the scene.
* **Status**: ✅ Available since Blueprint.

Choose a light with default properties to insert into the world, or attached
to an existing link. Support point, spot and directional lights.

#### B.5. Scaling tool

* **Modes**: Edit
* **Priority**: Must have
* **References**
* **User stories**
    * User wants to insert a box and 4 cylinders, scale them and assemble a simple vehicle.
* **Status**: ⌛

The first implementation can be done in edit mode so that the physics engine
doesn't need to react to size changes in real time (only on unpause).

### C. Entity creation

Features related to assembling or modifying a kinematic structure.

#### C.1. Create joints

* **Modes**: Edit
* **Priority**: Must have
* **References**
    * [Gazebo tutorial](http://gazebosim.org/tutorials?tut=model_editor#CreateJoints)
* **User stories**
    * User has several 3D meshes and wants to create an articulated model from it.
* **Status**: ⌛

Visual tool to aid creating joints between links.

Similar to Gazebo-classic's, a widget which offers several joint options and
lets the user click on links on the 3D scene to connect them.

#### C.2. Create sensors

* **Modes**: Edit / interactive
* **Priority**: Could have
* **References**
* **User stories**
    * User is assembling a mobile robot and wants to attach an RGB camera to it.
* **Status**: ⌛

This would add a sensor with default properties to an existing link. The user
would be able to choose the sensor type.

#### C.3. Attach plugins

* **Modes**: Edit / interactive
* **Priority**: Should have
* **References**
* **User stories**
    * User wants to add new functionality to an entity at runtime.
* **Status**: ⌛

Gazebo-classic's model editor supports adding plugins to the model, and the
user can add custom SDF configuration.

Ignition could support the same, for all entity types.

In the future, it would be nice if plugins could declare their SDF parameters
so the GUI can display all options to be filled.

#### C.4. Insert models

* **Modes**: Edit / Interactive
* **Priority**: Must have
* **References**
    * [Gazebo tutorial](http://gazebosim.org/tutorials?tut=guided_b2&cat=#LeftPanel)
* **User stories**
    * User has a specific SDF model saved in their local disk which they wish
      to insert into simulation.
    * User is creating an environment for their robot to move in, but they don't
      have any other models locally besides the robot.
* **Status**: ✅ Since Blueprint, it's already possible to spawn models by dragging from
https://app.ignitionrobotics.org.

In edit mode, entire models can be inserted as nested models, or broken apart
into links to be reused by other models.

In interactive mode, models are spawned into the world at runtime.

### D. Manipulation

Features related to changing the pose of shapes.

#### D.1. Translation and rotation tool

* **Modes**: Edit / Interactive
* **Priority**: Must have
* **References**
    * [Ignition Citadel: Transform Control](https://ignitionrobotics.org/docs/citadel/manipulating_models#transform-control)
* **User stories**
    * User wants to test the robot on a different place in the world, without
      restarting simulation.
* **Status**: ✅ Available since Acropolis.

#### D.2. Align

* **Modes**: Edit / Interactive
* **Priority**: Should have
* **References**
    * [Ignition Citadel: Align Tool](https://ignitionrobotics.org/docs/citadel/manipulating_models#align-tool)
* **User stories**
* **Status**: ✅ Available since Citadel.

#### D.3. Joint manipulation

* **Modes**: Edit / Interactive
* **Priority**: Could have
* **References**
* **User stories**
* **Status**: ⌛

Manual joint manipulation through its range of motion.

### E. Workflow

Features related to the user's workflow.

#### E.1. Property inspector

* **Modes**: Edit / Interactive
* **Priority**: Must have
* **References**
    * [Gazebo tutorial](http://gazebosim.org/tutorials?tut=model_editor#Edityourmodel)
    * [Ignition Citadel: Component Inspector](https://ignitionrobotics.org/docs/citadel/manipulating_models#component-inspector)
* **User stories**
    * User has an SDF model and wants to change the mass of a link.
    * User has an SDF light and wants to change its color.
    * User has an SDF model and wants to change the position of a joint.
* **Status**: ✅ Read-only inspector for several component types, as well as pose editing,
available from Blueprint.

More component types need to be made writable, according to the current
simulation mode.

#### E.2. 3D Visualizations

* **Modes**: Edit / Interactive
* **Priority**: Should have
* **References**
    * [Issue: Visualize inertia](https://github.com/ignitionrobotics/ign-gazebo/issues/111)
    * [Issue: Visualize center of mass](https://github.com/ignitionrobotics/ign-gazebo/issues/110)
    * [Issue: Visualize joints](https://github.com/ignitionrobotics/ign-gazebo/issues/106)
    * [Issue: Visualize collision shapes](https://github.com/ignitionrobotics/ign-gazebo/issues/105)
* **User stories**
    * User wants to debug a model spawned into simulation.
* **Status**: ⌛

Visualization of entities other than visuals, such as collisions, joints,
sensors, lights, etc.

Similar to Gazebo-classic, these can be reachable from a context menu.
Visibility options could also be added to the component inspector. Rendering
visuals will need to be implemented.

#### E.3. Undo / redo

* **Modes**: Edit / Interactive
* **Priority**: Should have
* **References**
    * [Gazebo design document](https://github.com/osrf/gazebo_design/blob/master/undo/undo.md)
* **User stories**
    * User wants to debug a model spawned into simulation.
* **Status**: ⌛

For interactive mode, user commands are processed by the server. The
[UserCommands](https://github.com/ignitionrobotics/ign-gazebo/tree/master/src/systems/user_commands)
system is already keeping commands in a queue. Completing
the implementation will involve:

* Implementing the undo / redo behavior for each command.
* Reverting time and world state.

For edit mode, the command queue can be completely handled by the client.

#### E.4. Save to SDF

* **Modes**: Edit / Interactive
* **Priority**: Must have
* **References**
    * [Issue: Save model states when saving the world](https://github.com/ignitionrobotics/ign-gazebo/issues/137)
    * [Issue: Right-click and save model](https://github.com/ignitionrobotics/ign-gazebo/issues/138)
* **User stories**
    * User has created a world by placing a lot of models and wants to reuse it later.
    * User has created a model and wants to use it later.
* **Status**: ✅ Saving the world is available since Blueprint.

Saving a world or parts of it to an SDF file.


#### E.5. Copy / paste

* **Modes**: Edit / Interactive
* **Priority**: Could have
* **References**
    * [Issue: Copy / paste entities from the GUI](https://github.com/ignitionrobotics/ign-gazebo/issues/102)
* **User stories**
    * User created a domino and wants to add several copies of the domino into
      the world to knock them down.
* **Status**: ⌛

Select one or more entities, then copy (Ctrl+C) and paste (Ctrl+V) somewhere else.

#### E.6. Schematic view

* **Modes**: Edit / Interactive
* **Priority**: Could have
* **References**
* **User stories**
* **Status**: ⌛

Display a graph with the parent-child relationships between entities in the world.

#### E.7 Simulation speed

* **Modes**: Interactive
* **Priority**: Could have
* **References**
* **User stories**
* **Status**: ⌛

TODO

#### E.8 Seek back in time

* **Modes**: Interactive
* **Priority**: Could have
* **References**
* **User stories**
* **Status**: ⌛

Let the user rewind the simulation to a given simulation time.

#### E.9 Physics engine selection

* **Modes**: Edit
* **Priority**: Could have
* **References**
* **User stories**
  * A user wants to try different physics engines at run time.
* **Status**: ⌛

TODO

#### E.10 Render engine selection

* **Modes**: Edit
* **Priority**: Could have
* **References**
* **User stories**
  * A user wants to try different render engines at run time.
* **Status**: ⌛

TODO

### F. Recording

#### F.1. Video creator

* **Modes**: Edit / Interactive
* **Priority**: Could have
* **References**
* **User stories**
* **Status**: ✅ Available since Blueprint.

#### F.2. Log record

* **Modes**: Interactive
* **Priority**: Must have
* **References**
    * [Ignition Gazebo: Logging](https://ignitionrobotics.org/api/gazebo/3.0/log.html)
* **User stories**
* **Status**: ✅ Available since Blueprint.

### G. Materials

#### G.1. Set materials on 3D

* **Modes**: Edit
* **Priority**: Could have
* **References**
* **User stories**
* **Status**: ⌛

TODO

### H. Introspection

#### H.1. Plotting

* **Modes**: Interactive
* **Priority**: Could have
* **References**
* **User stories**
* **Status**: ⌛

TODO

#### H.2. Topic viewer

* **Modes**: Interactive
* **Priority**: Could have
* **References**
* **User stories**
* **Status**: ⌛

TODO

