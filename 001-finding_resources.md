# Design Document 001 - Finding Resources

TODO: Overview

## Finding SDF URI Resources

The following sections discuss how `<uri>` elements in SDF files are
resovled in order to find meshes and models. This section does not cover
finding other resources, such as plugins.

### Current Process

This section describes how Ignition Gazebo currently finds Mesh URI
resources. SDF contains multiple instances of URI elements, but the current
implementation of Gazebo handles only mesh URIs. The information contained
here can be extrapolated to handle other types of resources, such as
textures and audio files.

A user of Ignition may not need or use Gazebo. If this describes you, then
skip directly to the last step which describes how
`ignition::common::findFile` can be used to resolve a resource.

1. Ignition Gazebo registers an SDF find callback in the Server constructor.
   This callback is triggered by SDF whenever a URI resource from an
   `<include>` element cannot be found. The Ignition Gazebo find callback in
   turn calls Fuel Tools to download the resource.

2. After registering the callback, Ignition Gazebo then finds the world file
   specified on the command line in a path contained in the
   `IGN_GAZEBO_RESOURCE_PATH` environment variable using
   `common::SystemPaths::FindFile`.

3. If the world file is found, then SDF is used to load the world file. SDF
   will only attempt to resolve URI resources contained within the world
   file that are in `<include>` elements.

4. A SimulationRunner is then instantiated for each `sdf::World`. The
   LevelManager, contained in each SimulationRunner, loads each `sdf::Model`
   and creates the appropriate set of components. Plugin loading is conducted
   during this step, but no mesh URI resource loading is conducted.

5. Systems and other aspects of simulation, such as rendering, must then each
   resolve URI resources contained within the loaded world and models.
   The `Util::asFullPath` is a helper function that is used to convert a URI
   into a system path. This helper function only alters URIs that have
   a relative path. URIs that have a scheme or are a full path are
   untouched.

6. The result of `Util::asFullPath` is then passed to
   `ignition::common::MeshManager::Load`, which then calls
   `ignition::common::findFile` to finally resolve the location of the URI
   resource.

7. `ignition::common::findFile` executes the following procedure.

    1. Calls `SystemPaths::FindFile(string file, bool searchLocalPath)`
    2. `SystemPaths::FindFile(string file, bool searchLocalPath)` executes:
        1. if `file` is empty, then return an empty string
        2. if `file` is a valid URI, then `path`
           = `SystemsPath::FindFileURI(file)`. The `FindFileURI(file)`
           executes:
            1. if the URI scheme is "file", then `path` = `SystemsPath::FindFile(_file)`. This will only be executed if `SystemsPath::FindFileURI(file)` is called indepently of `SystemPaths::FindFile`.
            2. else if there is a user define CB, the `path` = `FileURICB(_file)`. This `FileURICB` is deprecated in ign-common3.
            4. if `path` is empty, try each optional `FileURICbs`. The
               vector of `FileURICbs` replaces the deprecated `FileURICB`.
            5. if `path` is empty or `path` doesn't exist, then return empty string
            6. return `path`.
        3. else if `file` is an absolute file path, then `path` = `file`.
        4. else if `searchLocalPath == true` and the `file` exists in the
           current working directory, then `path` = current working
           directory + `file`.
        5. else if `file` is an absolute path and exists, then `path` = `file`
        6. else `path` = `findFileCB(file)`. `findFileCb` is an optional user
           defined callback.
        7. if `path` is empty, search the set of defined `filePaths`.
        8. if `path` is empty, call the optional user defined callbacks
        9. if `path` is empty or `path` does not exist, return an empty string.
        10. return `path`.

#### Issues with the current process

Problems with the current process exist in the last step, when `ignition::common::findFile` is executed.

The problems are:

1. Any scheme that is not "file", will not resolve unless a user defines
   a custom callback. The "model" scheme is particularly common. ROS makes
   use of the "package" scheme, which may sneak into SDF files. There is
   also no standard mechanism in Ignition Common to handle the "http[s]"
   scheme.

2. The behavior of custom schemes, especially the "model" scheme, are not
   defined. This has lead to different interpretations. Our, Open Robotics,
   desired interpretation of the "model" scheme is to reference a specific
   model whose name immediately follows the "model://" portion of a URI. Any
   remaining elements in the URI path wourld specify a resource relative to
   the named model. One unplanned use case is treat the "model" scheme as
   a "file" scheme. For example, a user could specify
   "model://relative/path/to/resource". In this case, Gazebo classic would
   search for the resource in the paths contained in the
   `GAZEBO_RESOURCE_PATH` environment variable.

3. The optional user defined callbacks are not sufficiently rich to support
   the diversity of use cases. Currently, the user defined callback are
   executed at one location in the resource resolution pipeline. This does
   not allow a user to completely override the default behavior and also
   implement a backup behavior.

4. Ignition Fuel Tools currently modifies URIs in downloaded models. The
   "model" scheme is converted to an absolute file path using the "file"
   scheme. This makes it easy to find resources, but also makes it difficult
   to modify and update a model.

## Finding Plugins

TODO

## Environment Variables

TODO
