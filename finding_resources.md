# Finding Resources

This document offers guidelines and rationales for the various mechanisms
used throughout Ignition to find resources (files) at runtime. File types
include, but are not limited to, plugins (shared libraries), world
descriptions (SDF files) and graphics files like meshes and textures.

This is not meant to list all the specifics of how Ignition finds each resource,
although it may mention some as examples. For an exhaustive explanation of how
each type of resource is found, please check
[Ignition Gazebo's Finding resources tutorial](https://ignitionrobotics.org/api/gazebo/3.0/resources.html).

**Table of Contents**

* [Ways of finding resources](#ways-of-finding-resources)
   * [Full path](#full-path)
   * [Full URL](#full-url)
   * [Relative paths](#relative-paths)
      * [Schemes](#schemes)
      * [Prefixes](#prefixes)
* [Precedence](#precedence)

## Ways of finding resources

The simplest way of getting to a resource is by having its **full path** on disk.
However, full paths are not portable when files need to be shared among users,
machines and platforms.

Another way is having the **full URL** of the resource when it's hosted online.

Finally, a common portable way of sharing paths is by making them relocatable.
That is, defining them relative to a movable location.

A couple of ways that this can be done are:

* Using relative paths
    * i.e. `path/to/some.file`
    * Requires conventions about what that's relative to, the running directory,
      the file's directory, etc.
* Using substitutable schemes
    * i.e. `special://path/to/some.file`
    * Gives each scheme the ability of defining its own substitution rules.

Below we will describe how Ignition uses each of these ways of finding.

### Full path

A full path is the least portable way of defining a file, but it has the
advantage of being straight-forward.

Ignition developers should be mindful of different path formats for different
platforms and support as many as possible. For example:

* Unix: `/absolute/path/to/file.ext`
* Windows: `C:\absolute\path\drive\c.ext`
* [File URIs](https://tools.ietf.org/html/rfc8089): `file:///absolute/path/tp/file.ext`

For example, users can use the full path when loading a world on Ignition Gazebo:

`ign gazebo /absolute/path/to/file.sdf`

We should strive to support loading **all kinds of resources** from their full
paths throughout the Ignition stack. That's not because we encourage users to
use this approach alone, but because it's helpful while debugging and in general
users expect that to work.

### Full URL

Ignition's official online database of resources is
[Ignition Fuel](https://app.ignitionrobotics.org/). The
[Ignition Fuel Tools](https://ignitionrobotics.org/libs/fuel_tools)
library provides a command line tool, as well as a C++ API to manage these
online resouces.

For example, users can use Fuel URLs when including a model into a world
to be loaded by Ignition Gazebo:

```
<include>
  <uri>https://fuel.ignitionrobotics.org/1.0/OpenRobotics/models/Bowl</uri>
</include>
```

Loading resources from the internet lets users pull from a common updated
source instead of having all the files they need on disk at all times.

Ignition developers should strive to support full Fuel URLs for **all resources
that are supported on the online platform**. The primary interface to the web
servers should be Ignition Fuel Tools. It's discouraged to make direct
REST calls to the server from within Ignition libraries because that would
give us a larger surface to maintain.

Developers shouldn't assume that every `http` scheme is coming from Fuel.
Instead, they should provide ways for downstream developers to register
callbacks to handle their own web servers.

### Relative paths

When a path is relative, it means that it must be prefixed by an absolute
path before loading.

#### Schemes

Ignition's internal mechanisms will treat all schemes the same way, by
stripping them from the path. However, downstream libraries are welcome to
handle special schemes as they wish.

That means that for Ignition, the following are equivalent:

* `model://my_model/meshes/shape.stl`
* `package://my_model/meshes/shape.stl`
* `banana://my_model/meshes/shape.stl`

Once the scheme is stripped, the paths will look like simple relative paths:

* `my_model/meshes/shape.stl`

Ignition developers should always fallback to stripping schemes after failing
to load resources with a scheme, **for all resource types**.

#### Prefixes

**Installation path**

Libraries may install resources such as plugins and worlds that can be found at
runtime. In general, libraries that handle a given resource type should
automatically find **all resources of that type that are installed by all
Ignition libraries**.

For example, users can load GUI plugins installed with both Ignition Gazebo and
Ignition GUI by referring to their shared library's filename stripped of the
`lib` prefix and the extension.

On a Linux system with Ignition GUI installed from binaries, this
example loads both
`/usr/lib/x86_64-linux-gnu/ign-gui-2/plugins/libWorldControl.so` and
`/usr/lib/x86_64-linux-gnu/ign-gazebo-2/plugins/gui/libEntityTree.so`.

```
<plugin filename="WorldControl" name="My world control">
</plugin>
<plugin filename="EntityTree" name="My entity tree">
</plugin>
```

Loading from installation paths gives users a nice out-of-the-box experience.

**Environment variables**

Predefined environment variables may hold a list of colon-separated paths that
can be used as path prefixes.

For example, a user can create a custom physics engine plugin and set
`IGN_GAZEBO_PHYSICS_ENGINE_PATH` to the library's parent path. Then they can
load the library just using its filename without `lib` or extension. On a
Linux system, the example below will load the plugin located at
`/home/physics_engines/libCustomEngine.so`.

```
export IGN_GAZEBO_PHYSICS_ENGINE_PATH=/home/physics_engines
ign gazebo --physics-engine CustomEngine
```

Reading paths from environment variables gives users and packages a quick way
to add support for custom paths.

**All resource types** should be searchable by at least one environment
variable.

**Current directory**

When running an application from the command line, often users expect that
resources can be described with respect to that directory.

For example, users can use a path relative to the current directory when
loading an Ignition Launch file:

`ign launch relative/path/to/file.ign`

Similar to absolute paths, the use of paths relative to the current working
directory can be handy for users.

**All resources that are loaded from the command line** must support paths
relative to the current directory. This is optional for files that are
referenced from other files, like textures.

**Current file path**

When a resource is referenced from another file, it's common to treat the
path to the referenced file as relative to the original file.

For example, given an SDF model with the following structure:

```
MyModel
├── meshes
│   └── my_mesh.dae
├── model.config
└── model.sdf
```

The `model.sdf` file can refer to `my_mesh.dae` as:

```
<uri>meshes/my_mesh.dae</uri>
```

Accepting relative paths within files makes it convenient to work with
resources that consist of several files that reference each other.

**All resources that are references from another file** must support paths
relative to the original file.

**Common home directory**

Directories under `$HOME/.ignition/<library_name>` can also be used as a default
search path. This location is convenient because it doesn't require special
permissions and is always the same for all users. It's also a good place to put
default files that the user should be allowed to customize.

For example, when loading Ignition Gazebo, by default the GUI configuration
is taken from:

`$HOME/.ignition/gazebo/gui.config`

The initial file on that location is copied from the `gui.config` that is
installed with Ignition Gazebo. But the user can edit it to change the style
of their windows without affecting other users.

Another example is the `$HOME/.ignition/gazebo/plugins` directory, where Ignition
Gazebo will always look for system plugins.

A central location is useful for users who just want to put their files in a
location where they will always be found with minimal setup.

**All resource types** should have at least one search path under `~/.ignition`.

## Precedence

There are many ways of referring to a file. Likewise, one path may match
several files.

For example, let's assume that all these files exist:

```
/tmp/worlds/shapes.sdf
/tmp/more_worlds/shapes.sdf
/home/user/shapes.sdf
/usr/share/ignition/ignition-gazebo2/worlds/shapes.sdf
```

When the user does the following, which file gets loaded?

```
cd /home/user
IGN_GAZEBO_RESOURCE_PATH=/tmp/worlds:/more_worlds
ign gazebo shapes.sdf
```

It's necessary to have a consistent way of prioritizing which approach
preceeds others, so that the user knows what to expect.

The general guideline for precedence is to go **from the most custom to the most
standard**. The most custom methods can be easily changed by the user at runtime,
while the least custom ones would require recompiling a library.

On the example above, it's most likely safe to assume that the user
didn't want to load the world that is installed with Ignition Gazebo, otherwise
they wouldn't have gone through the trouble of moving directory and setting
an environment variable. But in the end, we can never know for sure what the
user really meant. They could happen to be in that directory, and
the variable may have been set by another application that appended it to
`~/.bashrc`.

That is to say that there's no perfect solution. But at least by keeping
consistency users can refer back to these guidelines and understand
what to expect.

Thus, Ignition should look for resources in the following order:

1. Environment variables
2. Path relative to current directory
3. Full path
4. Common home directory
5. Custom callbacks set by downstream developers
6. Fuel URL (Fuel checks cache first)
7. Resources installed with Ignition

