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
happen any time. In other words, whenever simulation is **paused**,
**edit mode** will be enabled.

Different modes of operation are common among software which allows both
creating and running content. Think about how slides can't be edited while
they're being presented, or how video editors will show short previews while
editing a video, but once the video is saved it can no longer be edited. Closer
to the robotics world, we have game engines which often have a clearly
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

## User stories

1. User has a 3D mesh and wants to create a rigid model from it.
2. User has several 3D meshes and wants to create an articulated model from it.
3. User has a 2D image and wants to extrude that as a geometry for a model.
4. User has an SDF model and wants to change the mass of a link.
4. User has an SDF light and wants to change its color.
4. User has an SDF model and wants to change the position of a joint.

## Features

### User camera controls

#### Mouse orbiting

**Need**:
**User stories**:
**Effort**:
**References**:

#### Preset view angles

**Need**:
**User stories**:
**Effort**:
**References**:

#### Move to entity

**Need**:
**User stories**:
**Effort**:
**References**:

#### Follow entity

**Need**:
**User stories**:
**Effort**:
**References**:

### Geometry creation

#### Import 3D meshes

Import a mesh file from the local file system to use as a visual or collision
geometry.

**Need**: Required
**User stories**: 1
**Effort**:
**References**:
* [Gazebo tutorial](http://gazebosim.org/tutorials?tut=model_editor#Addmeshes)

#### Extrude 2D shapes

Extrude a 2D file from the local file system into a 3D shape to use as a visual
or collision geometry.

**Need**: Nice to have
**User stories**: 3
**Effort**:
**References**:
* [Gazebo tutorial](http://gazebosim.org/tutorials?tut=extrude_svg)

#### Insert simple shapes

Choose a primitive shape to use as a visual or collision geometry. Support
specifying parameters such as size, physical material and visual material.

**Need**: Nice to have
**User stories**: 3
**Effort**:
**References**:
* [Gazebo tutorial](http://gazebosim.org/tutorials?tut=model_editor#Addsimpleshapes)

### Kinematics

#### Create joints

Visual tool to aid creating joints between links.

**Need**: Required
**User stories**:
**Effort**:
**References**:
* [Gazebo tutorial](http://gazebosim.org/tutorials?tut=model_editor#CreateJoints)

---

TODO: Categorize these:

#### View collisions

**Need**:
**User stories**:
**Effort**:
**References**:

#### View joints

**Need**:
**User stories**:
**Effort**:
**References**:

#### Property inspector

**Need**:
**User stories**:
**Effort**:
**References**:
* [Gazebo tutorial](http://gazebosim.org/tutorials?tut=model_editor#Edityourmodel)

#### Copy / paste

**Need**:
**User stories**:
**Effort**:
**References**:

#### Undo / redo

Commands processed during edit mode can be completely handled by the client.

**Need**:
**User stories**:
**Effort**:
**References**:

#### Align

**Need**:
**User stories**:
**Effort**:
**References**:

#### Schematic view

**Need**:
**User stories**:
**Effort**:
**References**:

#### Save SDF

**Need**:
**User stories**:
**Effort**:
**References**:

#### Manipulate entities in 3D

Support manual manipulation, such as moving a joint through its range of motion.

**Need**:
**User stories**:
**Effort**:
**References**:


### Recording

#### Video creator

**Need**:
**User stories**:
**Effort**:
**References**:

#### Log record

**Need**:
**User stories**:
**Effort**:
**References**:

## Lessons from Gazebo-classic

* Exiting the model editor is time consuming because we always save a new SDF
model. If we'll be using the play button as a toggle for edit mode, we need to
do something else so we don't hang every time the user unpauses.

* The different interfaces for visualizing properties during simulation and
during editing caused confusion for users.




---

# TODO: Add these

Sensor property visualization (such as camera frustum and lidar range) and parameter modification
Attach sensors
Attach plugins
Insert models from Fuel

Interactive mode only:

* Control simulation speed
* Log record and playback ?
* Plotting
* Topic viewer
* Sensor data visualization
