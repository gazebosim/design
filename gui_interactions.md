# GUI interactions

This document is meant to inform and guide the development of various GUI
interactions for Ignition Gazebo.

## Overview

Ignition Gazebo's graphical interface's high-level goals are to:

1. **Introspect** a running simulation
2. **Interact** with a running simulation
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
interaction. Even actions like changing mass or adding a new link will be
helpful to some user. However, it's important to keep in mind that handling
interactions is considerably more complicated than handling editing, primarily
because physics engines are usually designed to deal with nodies that have
immutable kinematics and dynamics.

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

## Features

### User camera controls

#### Mouse orbiting

* **Modes**: Edit / Interactive
* **Priority**: Must have
* **References**
    * TODO: link to Ignition tutorial
* **User stories**
    * User wants to look at different parts of the world.

Available since Acropolis.

#### Preset view angles

* **Modes**: Edit / Interactive
* **Priority**: Should have
* **References**
    * TODO: link to Ignition tutorial
* **User stories**
    * User is constructing a world and wants to check how models are aligned from
      different angles.

Available since Blueprint.

#### Move to entity

* **Modes**: Edit / Interactive
* **Priority**: Should have
* **References**
    * TODO: link to Ignition tutorial
* **User stories**
    * User is inspecting a large world, and they want to find a specific
      entity.

Available since Blueprint.

#### Follow entity

* **Modes**: Interactive
* **Priority**: Should have
* **References**
    * TODO: link to Ignition tutorial
* **User stories**
    * User is simulating a mobile robot and wants to keep it in view at all
      times.

Available since Blueprint.

### Geometry creation

Features related to creating and importing geometric shapes to be used
as rigid bodies in simulation.

#### Import 3D meshes

* **Modes**: Edit
* **Priority**: Must have
* **References**
    * [Gazebo tutorial](http://gazebosim.org/tutorials?tut=model_editor#Addmeshes)
        * Gazebo-classic's default behavious is to create a link and use the
          mesh as geometry for both visual and collision.
* **User stories**
    * User has a 3D mesh and wants to create a rigid model from it.

Import a mesh file from the local file system to use as a visual or collision
geometry.

Ignition supports DAE, OBJ and STL meshes. Implementation of the importing
widget would involve opening a file browser and spawning the selected file into
simulation.

#### Extrude 2D shapes

* **Modes**: Edit
* **Priority**: Could have
* **References**
    * [Gazebo tutorial](http://gazebosim.org/tutorials?tut=extrude_svg)
* **User stories**
    * User has a 2D image and wants to extrude that as a geometry for a model.

Extrude a 2D file from the local file system into a 3D shape to use as a visual
or collision geometry.

Like on Gazebo-classic, the widget could load an SVG file, let the user choose
a few parameters, and spawn the shape as collision and visual. Polyline support
will also need to be added.

#### Insert simple shapes

* **Modes**: Edit / Interactive
* **Priority**: Must have
* **References**
    * [Gazebo tutorial](http://gazebosim.org/tutorials?tut=model_editor#Addsimpleshapes)
* **User stories**
    * User wants to create a simple vehicle composed of boxes and cylinders.

Choose a primitive shape to use as a visual or collision geometry. Support
specifying parameters such as size, physical material and visual material.

The widget would offer a few simple shape options (box, cylinder, sphere). It
could have additional options on a dropdown, such as material, which would
define the mass, inertia, and perhaps even visual appearance.

### Entity creation

Features related to assembling or modifying a kinematic structure.

#### Create joints

* **Modes**: Edit
* **Priority**: Must have
* **References**
    * [Gazebo tutorial](http://gazebosim.org/tutorials?tut=model_editor#CreateJoints)
* **User stories**
    * User has several 3D meshes and wants to create an articulated model from it.

Visual tool to aid creating joints between links.

Similar to Gazebo-classic's, a widget which offers several joint options and
lets the user click on links on the 3D scene to connect them.

#### Create sensors

* **Modes**:
* **Priority**:
* **References**
* **User stories**

TODO

#### Attach plugins

* **Modes**:
* **Priority**:
* **References**
* **User stories**

TODO

#### Insert models

* **Modes**: Edit / Interactive
* **Priority**: Must have
* **References**
    * [Gazebo tutorial](http://gazebosim.org/tutorials?tut=guided_b2&cat=#LeftPanel)
* **User stories**
    * User has a specific SDF model saved in their local disk which they wish
      to insert into simulation.
    * User is creating an environment for their robot to move in, but they don't
      have any other models locally besides the robot.

In edit mode, entire models can be inserted as nested models, or broken apart
into links to be reused by other models.

In interactive mode, models are spawned into the world at runtime.

### Manipulation

Features related to changing the pose of shapes.

#### Translation and rotation tool

* **Modes**: Edit / Interactive
* **Priority**: Must have
* **References**
    * TODO: link to Ignition tutorial
* **User stories**
    * User wants to test the robot on a different place in the world, without
      restarting simulation.

Available since Acropolis.

#### Align

* **Modes**: Edit / Interactive
* **Priority**: Should have
* **References**
    * TODO: link to Ignition tutorial
* **User stories**

Available since Blueprint.

#### Joint manipulation

* **Modes**: Edit / Interactive
* **Priority**: Could have
* **References**
* **User stories**

TODO

Manual joint manipulation through its range of motion.

### Workflow

Features related to the user's workflow.

#### Property inspector

* **Modes**: Edit / Interactive
* **Priority**: Must have
* **References**
    * [Gazebo tutorial](http://gazebosim.org/tutorials?tut=model_editor#Edityourmodel)
    * TODO: link to Ignition tutorial
* **User stories**
    * User has an SDF model and wants to change the mass of a link.
    * User has an SDF light and wants to change its color.
    * User has an SDF model and wants to change the position of a joint.

Read-only inspector for several component types available from Blueprint.
Component inspector will need to be updated to support edit mode and enable
writable widgets for components which can be saved to SDF.

#### 3D Visualizations

* **Modes**: Edit / Interactive
* **Priority**: Should have
* **References**
* **User stories**

Visualization of entiites other than visuals, such as collisions, joints,
sensors, lights, etc.

Similar to Gazebo-classic, these can be reachable from a context menu.
Visibility options could also be added to the component inspector. Rendering
visuals will need to be implemented.

#### Undo / redo

* **Modes**: Edit / Interactive
* **Priority**: Should have
* **References**
    * [Gazebo design document](https://bitbucket.org/osrf/gazebo_design/src/default/undo/undo.md)
* **User stories**

For interactive mode, user commands are processed by the server. The
`UserCommands` system is already keeping commands in a queue. Completing
the implementation will involve implementing the undo / redo behaviour for each
command. There's also the need to revert time and world state.

For edit mode, the command queue can be completely handled by the client.

#### Save to SDF

* **Modes**: Edit / Interactive
* **Priority**: Must have
* **References**
* **User stories**

Saving a world or parts of it to an SDF file.

#### Copy / paste

* **Modes**: Edit / Interactive
* **Priority**: Could have
* **References**
* **User stories**

TODO

#### Schematic view

* **Modes**: Edit / Interactive
* **Priority**: Could have
* **References**
* **User stories**

TODO

#### Simulation speed

* **Modes**: Interactive
* **Priority**: Could have
* **References**
* **User stories**

TODO

### Recording

#### Video creator

* **Modes**: Edit / Interactive
* **Priority**: Could have
* **References**
* **User stories**

TODO

#### Log record

* **Modes**: (Edit?) / Interactive
* **Priority**: Must have
* **References**
* **User stories**

TODO

### Materials

#### Set materials on 3D

* **Modes**: Edit
* **Priority**: Could have
* **References**
* **User stories**

TODO

### Introspection

#### Plotting

* **Modes**: Interactive
* **Priority**: Could have
* **References**
* **User stories**

TODO

#### Topic viewer

* **Modes**: Interactive
* **Priority**: Could have
* **References**
* **User stories**

TODO

## Lessons from Gazebo-classic

* Exiting the model editor is time consuming because we always save a new SDF
model. If we'll be using the play button as a toggle for edit mode, we need to
do something else so we don't hang every time the user unpauses.

* The different interfaces for visualizing properties during simulation and
during editing caused confusion for users.

