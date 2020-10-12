# Tape Measure Tool

This document outlines the design for the tape measure tool targeted at
Ignition Dome and onwards.  The reasoning for the targeting of Dome is largely
due to the snapping behavior which needs bounding boxes.  If this measurement
tool is needed for Citadel, a more simple version that doesn't contain
any of the more complex snapping capabilities can be considered.

## Purpose

The tape measure tool, as its name implies, is a tool that aids the user in
measurements of distance within a 3D scene. Currently, there is no method
of extracting any sort of measurement data.  This sort of information is data
that is quite useful to a user working within Ignition Gazebo, whether it be
making various calculations or designing/creating a world and needing exact
precision.  It can also just be simply because it is easy to lose the scale of
the world and the objects within it when working within a simulator. This
functionality will also be quite convenient with the eventual addition of
entity scaling.  On a tangent, the dimensional data of an entity, which will
likely be a reason for which this tool will be used the most, could also
likely be placed within the Component Inspector plugin so the user isn't left
clicking all of the bounds of an entity and having to manually keep track of
the entity's dimensions themselves.  There is an issue out for similar type
improvements [here](https://github.com/ignitionrobotics/ign-gazebo/issues/158)
where this data is expected to eventually be added.

## Functionality

The tape measure tool is likely to be very simplistic in nature, but has
potential to have certain nuances that this design doc is intended to flesh
out.

The user experience should be similar to the following:
 1. The user adds the Tape Measure Tool plugin via the plugin menu.
 2. At this point, there will be a button one could click that is something
    like "New measurement" at which point would enable the tape measure tool.
    There will also be a hotkey to start a new measurement so the user can
    quickly take measurements without having to constantly click the
    "New Measurement" button.
 3. The mouse will then change into a cross-shaped icon and display a
    transparent point which gives a preview of the point to be measured,
    this point will become opaque once the user clicks.
 4. The user will click point A and then click point B.  This method allows
    for navigation within the scene in between the choosing of the points.
 5. When the user makes their selection, the markers should appear at point
    A and point B, and the user should see a marker line between the two
    points, even while being in the process of choosing point B.
 6. The distance will be written in text along the line marker that appears
    between the two points as well as written in a text box within the plugin
    window.
 7. At any point during this process, the user can right click to cancel
    the measurement.
 8. The user can hold control to snap to certain key points such as the
    boundaries of an entity's bounding box, the corners of those bounding
    boxes, or the grid lines as well.

## UI Design

This design isn't too complicated by nature.  There is some potential to even
add this tool as a default button residing next to the translation tools on
the top bar so UI simplicity is desired. A simple mock-up is as follows.  The
yellow color of the tape measure was simply chosen as it was the most
contrasting color in the shapes scene, but this color is likely to change.
The "New Measurement" rectangle is a button that the user can press to
initiate a new measurement.  It may also be helpful to display an xyz axis at the
origin of point A.

![Tape measure UI](images/tape_ui.png)

## Nice to haves

 * Display bounding boxes when hovering during the selection.
 * Manipulating the end points once the tape measure is placed.
 * Specifying the length of the tape so that it will lock to that length.
 * Specifying an angle relative to the x, y, or z axis that the
   tape measure will lock to.
 * Hold shift to snap and constrain the tape measure to the closest axis.
 * Multiple measurements within the scene
 * Attach a tape measure end point to a model/link if the user clicks one.
   See [Louise's Gazebo Classic example here](https://www.youtube.com/watch?v=XjszkNSthok&feature=youtu.be)
