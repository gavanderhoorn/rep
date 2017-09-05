REP: 199
Title: Coordinate Frames for Industrial Manipulators
Author: G.A. vd. Hoorn <g.a.vanderhoorn@tudelft.nl>
Status: Draft
Type: Informational
Content-Type: text/x-rst
Requires: 103
Created: 15-May-2017


Abstract
========

This REP specifies naming conventions and semantics for coordinate frames of industrial manipulators used with ROS.


Motivation
==========

Developers of packages aimed at industrial manipulators need a shared convention for coordinate frames in order to better integrate and re-use software components.
Consistent naming of frames and standardisation of semantics allow re-use of not only software but also of experience and tooling.

In addition, industrial robot controllers already define a number of Cartesian coordinate frames for users to define poses in and to refer to when manually controlling the robot (i.e.: while jogging or programming using teach-in).
In order to make it possible for ROS applications to make use of these coordinate frames in a controller- and vendor-agnostic way, this REP proposes a set of generalised frame names with associated semantics that allow for a correspondence between controller internal frames and the (TF) frames that a ROS application communicating with it might use.

REP 105 [#REP105]_ and REP 120 [#REP120]_ already define frames for mobile bases and humanoid robots.
This REP extends the set of standardised frames in ROS and labels important locations on industrial manipulators such as the origin of the main Cartesian coordinate system, the flange and the tool centre point.


Assumptions
===========

#. The key words *must*, *must not*, *required*, *shall*, *shall not*, *should*, *should not*, *recommended*, *may*, and *optional* in this document are to be interpreted as defined in RFC-2119 [#RFC2119]_.


Specification
=============

Coordinate Frames
-----------------

base_link
'''''''''

The coordinate frame called ``base_link`` shall be rigidly attached to the robot root body.
For industrial manipulators, this frame is expected to be located at the bottom of the first link - the base of the robot, or the surface that forms the robot's base mount - and forms the root of the kinematic chain of the manipulator in ROS.

Note that this frame does not need to be coincident with the origin of the controller's Cartesian frame, but rather it should be located in the logical base position of the robot.

This frame shall follow ROS conventions for orientation of frames as set forth in REP 103 [#REP103]_.
Discrepancies between a controller's internal Cartesian frame and REP 103 [#REP103]_ shall be resolved by defining the correct transform between ``base_link`` (or any other convenient frame) and the ``base`` frame (see `base`_).

``base_link`` may have geometry assigned to it, but this is not required.

No further special treatment of this frame is required, but making it the root of the kinematic chain in ROS is strongly recommended.

Rationale: this is the standard ROS ``base_link`` frame, but used in the context of industrial robots. As this frame is not required to be coincident with any frame of the robot controller, it can and should follow chirality and axes orientation conventions from REP 103 [#REP103]_.


base
''''

The ``base`` frame shall be coincident with the default Cartesian coordinate system as defined by the industrial robot controller.
Its purpose is to allow users to transform poses from a ROS application into the Cartesian base frame of the robot.
As such, this frame is exempt from the requirement to follow orientation conventions as described in REP 103 [#REP103]_.

Examples of vendor-specific names for this frame are *World* (Fanuc), *ROBROOT* (KUKA) and *Base* (ABB, Denso, Mitsubishi and Yaskawa Motoman).

Any frame is acceptable as the parent of ``base`` (so not just ``base_link``), as long as the transform between parent and ``base`` is fixed (i.e.: not across a movable joint).

``base`` shall not have any geometry associated with it.

Finally: ``base`` may be part of the (main) kinematic chain, but this is not required.
Frame hierarchies must however support transforms from anywhere in the chain to ``base`` and vice-versa (for example: from ``base`` to ``tool0``, or from ``link_4`` to ``base``).

Rationale: for many industrial robots, the location of the ROS ``base_link`` frame is not necessarily coincident with the origin of their main Cartesian coordinate system.
This complicates expressing poses in robot-relative frames, as arbitrary transformations could be required for different makes and models of robots.
The ``base`` frame is intended to be used as an intermediary or bridge and provides a mapping that links the ROS TF tree with the controller- and vendor-specific coordinate systems and as an abstraction that decreases coupling between applications and specific robot urdfs.


flange
''''''

The ``flange`` frame is the frame that should be used to attach EEF models to the main kinematic chain of the robot.
In contrast to ``tool0``, this frame shall always be oriented according to REP 103 [#REP103]_, with the positive axis (x+) pointing forward.
This makes attaching EEF models straightfoward, as no additional rotations should be needed to align the EEF model with the robot flange link.

Any frame is acceptable as the parent of ``flange``, as long as the transform between that parent and ``flange`` is fixed (i.e.: not across a movable joint), and ``flange`` is located in the correct location and has the correct orientation.
It is expected that in most cases ``flange`` will be a child of the last physical link of a robot's kinematic chain (ie: the 6th or 7th link for a standard industrial serial manipulator).

Some vendor-specific names for the tool frame are *FLANGE* (KUKA), TODO: finish.

``flange`` shall not have any geometry associated with it.

Shall not be changed.

Rationale: this separates the (physical) attachment point for EEFs from the mathematical TCP frame (which don't necessarily have to coincide for all robots, and also don't need to have the same orientation).


tool0
'''''

The ``tool0`` frame (pronounced: "tool-zero") shall match exactly an *all-zeros* Tool Centre Point (TCP) as defined by the robot controller.
As such, this frame is exempt from the requirement to follow orientation conventions as described in REP 103 [#REP103]_.
For most controllers, an all-zeros TCP is equal to an unconfigured (or default) TCP, which typically lies on the robot's physical mounting flange.
In this case the only difference between ``tool0`` and ``flange`` is the orientation.

Some vendor-specific names for the tool frame are *Tool Frame* (Fanuc), *TOOL* (KUKA), TODO: finish.

Any frame is acceptable as the parent of ``tool0``, as long as the transform between that parent and ``tool0`` is fixed (i.e.: not across a movable joint), and ``tool0`` is located in the correct location and has the correct orientation.
It is expected that in most cases ``tool0`` will be a child of the last physical link of a robot's kinematic chain (ie: the 6th or 7th link for a standard industrial serial manipulator).

``tool0`` must not be changed.
If application-specific tool frames are needed, these should be added as siblings of ``tool0``, or as part of a(n) (stand-alone) EEF definition attached to the ``flange`` frame.

``tool0`` shall not have any geometry associated with it.

Rationale: 

By not allowing changes to the location and orientation of ``tool0``, re-use of libraries such as kinematics solvers that are generated in an off-line fashion for a particular kinematic chain configuration becomes feasible.
It is the user's responsibility then to make sure that poses are transformed into the appropriate coordinate system before passing them on to such libraries.
This could be done automatically by the motion planner, or manually before submitting goal poses to the planner.


Dual or Multi-arm robots
------------------------

This REP does not specify any special conventions for robots with multiple arms/groups or kinematic chains.
The standard approach of prefixing joint and link names to ensure uniqueness of all frames in the (combined) frame hierarchy should be used to avoid collisions.
See the `Example Frame Hierarchies`_ section for examples of this.


Frame Authorities
-----------------

The frames described in this REP will typically be part of the static description of robot models encoded in urdfs or xacros.
As such, the frame authority is expected to be an instance of ``robot_state_publisher``, but this is not required.
In cases where (complicated) kinematics preclude the use of standard nodes, a specialised node capable of publishing the necessary frames could be used.


Exceptions
----------

The scope of potential robotics software is too broad to require all ROS software to follow the guidelines of this REP.
However, choosing different conventions should be well justified, well documented, and is discouraged.


Example Frame Hierarchies
=========================

Single manipulator
------------------

The following shows an example frame hierarchy for a single serial manipulator with just the default tool frame::

  base_link
  ├ base
  └ link_1
    └ link_2
      └..
       └ link_N
         ├ tool0
         └ flange

Single manipulator with EEF
---------------------------

The following shows an example frame hierarchy for a single serial manipulator with an EEF model attached to ``flange`` and an additional tool frame::

  base_link
  ├ base
  └ link_1
    └ link_2
      └ ..
        └ link_N
          ├ tool0
          ├ flange
          │ └ eef_base_link
          │   └ ..
          │     └ eef_link_N
          └ eef_tcp

Note the ``eef_`` prefix on the links in the EEF subhierarchy to prevent name clashes.

Dual manipulator
----------------

The following shows an example frame hierarchy for a work cell that consists of two manipulators::

  base_link
  ├ ..
  ├ left_base_link
  │ ├ left_base
  │ └ left_link_1
  │   └ left_link_2
  │     └..
  │      └ left_link_N
  │        ├ left_tool0
  │        └ left_flange
  ├ ..
  └ right_base_link
    ├ right_base
    └ right_link_1
      └ right_link_2
        └..
         └ right_link_N
           ├ right_tool0
           └ right_flange


Compliance
==========

This REP depends on and is compliant with REP 103 [#REP103]_, except where stated otherwise.


Questions
=========

#. are there robots that do not have ``tool0`` defined as a child of ``flange`` (I seem to remember some brand having ``tool0`` defined relative to the last link/joint, which was not the flange)? If not: make ``tool0`` child of ``flange``. Otherwise keep ``tool0`` a sibling of ``flange``.
#. should we discourage naming additional frames ``toolN`` (ie: ``tool1``, ``tool2``, etc)? Such names carry no semantics (frame purpose only known out-of-band), and all names in ROS should be as descriptive as possible.


References
==========

.. [#REP103] REP 103, Standard Units of Measure and Coordinate Conventions
   (http://www.ros.org/reps/rep-0103.html)

.. [#REP105] REP 105, Coordinate Frames for Mobile Platforms
   (http://www.ros.org/reps/rep-0105.html)

.. [#REP120] REP 120, Coordinate Frames for Humanoids Robots
   (http://www.ros.org/reps/rep-0120.html)

.. [#RFC2119] Key words for use in RFCs to Indicate Requirement Levels, on-line, retrieved 5 October 2015
   (https://tools.ietf.org/html/rfc2119)

.. [4] tool0: ROS-I vs industrial controllers
   (https://github.com/ros-industrial/ros_industrial_issues/issues/24)

.. [5] Fix for issues #49 and #95: ros-i compatible base and tool0 frames
   (https://github.com/ros-industrial/universal_robot/pull/200#issuecomment-102980913)

.. [6] Create a URDF for an Industrial Robot
   (http://wiki.ros.org/Industrial/Tutorials/Create%20a%20URDF%20for%20an%20Industrial%20Robot)

.. [7] Create a MoveIt Package for an Industrial Robot
   (http://wiki.ros.org/Industrial/Tutorials/Create_a_MoveIt_Pkg_for_an_Industrial_Robot)

.. [8] Working with ROS-Industrial Robot Support Packages
   (http://wiki.ros.org/Industrial/Tutorials/WorkingWithRosIndustrialRobotSupportPackages)


Copyright
=========

This document has been placed in the public domain.



..
   Local Variables:
   mode: indented-text
   indent-tabs-mode: nil
   sentence-end-double-space: t
   fill-column: 70
   coding: utf-8
   End:
