# Gazebo Log Format Update

## Overview

The Gazebo log format is an SQLite3 database that stores serialized protobuf
messages along with topic names in a time sequence. Importantly, simulation
state information is recorded as serialized Components. This relies on
a unique type id to associate the data to a component type. This type
information is available via c++ typeid, which is not readily
cross-language. For example, a javascript logfile reader would not know how
to associate a typeid with the correct component.


State information encoded as components requires a log file reader to
implement or use Gazebo’s ECS. This represents a large barrier to entry.
Ideally, a user could query data from a log file without implementing
a complete ECS. 

## Requirements

1. Cross-platform and cross-language
    1. Logged data
        1. The current log file format is dependent on C++ typeid when it stores serialized ECS Component data. Typeid information is not available in other languages, such as javascript.
        1. It’s reasonable to assume that a language can use Protobuf messages.
    1. Log file format
        1. The mechanism used to record data should also be cross platform.
1. Ease of use
    1. Queryable
        1. Support random access to logged data
    1. Low barrier to entry
        1. A user of the current log format needs to re-implement the Gazebo ECS.
1. Efficient
    1. Forward and backward iterators through the log file should be fast.
    1. It should be possible to quickly jump to a specific time in the log
       file.
    1. Low overhead when writing log files.
1. Self-contained
    1. Capture initial simulation state
    1. Capture topic names, message types, and message definitions
    1. Capture metadata such as
        1. Log format version
        1. Gazebo version
        1. Date and time of recording
    1. Optionally include all assets used by simulation. 
        1. This can be opt-out. Opting out indicates that log replay would rely on Fuel assets or locally available assets.
1. Streamable
    1. Do not require the entire database to be loaded before it can be read. This is very useful for web applications.
