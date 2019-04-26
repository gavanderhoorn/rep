REP: 199
Title: Coordinate Frames for Serial Industrial Manipulators
Author: G.A. vd. Hoorn <g.a.vanderhoorn@tudelft.nl>
Status: Draft
Type: Informational
Content-Type: text/x-rst
Requires: 103
Created: 11-Jan-2019


Abstract
========

This REP specifies naming conventions and semantics for coordinate frames and links of serial industrial manipulators that are used with ROS.


Motivation
==========

Developers of packages aimed at serial industrial manipulators need a shared convention for coordinate frames in order to better integrate and re-use software components.
Consistent naming of frames and standardisation of semantics allows for re-use of not only software but also of experience and tooling.

In addition, commercially available industrial robot controllers already define a number of Cartesian coordinate frames for users to define poses in and to refer to when manually controlling the robot (i.e.: while jogging or programming using teach-in).
In order to make it possible for ROS applications to refer to these coordinate frames in a controller and vendor-agnostic way, this REP proposes a set of generalised frame names with associated semantics that allow for a correspondence between controller internal frames and the (TF) frames that a ROS application communicating with it might use.

REP 105 [#REP105]_ and REP 120 [#REP120]_ already define frames for mobile bases and humanoid robots.
This REP extends the set of standardised frames in ROS and labels important locations on serial industrial manipulators such as the origin of the main Cartesian coordinate system, the flange and the tool centre point.
Finally, it provides naming guidelines for intermediate link frames and application-specific tool frames.


Definitions
===========

End Effector (EEF)
    A device at the end of a robotic arm, designed to interact with the environment.
    The exact nature of this device depends on the application of the robot.
Tool Centre Point (TCP)
    Point in relation to which all robot positioning is defined or the origin of the tool's coordinate frame.


Assumptions
===========

#. The key words *must*, *must not*, *required*, *shall*, *shall not*, *should*, *should not*, *recommended*, *may*, and *optional* in this document are to be interpreted as defined in [#RFC2119]_.


Specification
=============

Coordinate Frames
-----------------

base_link
'''''''''

The coordinate frame called ``base_link`` shall be rigidly attached to the robot root body.
For industrial manipulators, this frame is expected to be located at the bottom of the first link - the base of the robot, or the surface that forms the robot's base mount (ie: its attachment to the world).
If a robot is mounted with a different part to its surroundings, it is acceptable to place ``base_link`` in a different location.

Note that this frame is not required to be coincident with the origin of the controller's Cartesian frame, but rather it should be located in the logical base position of the robot.

This frame shall follow ROS conventions for both chirality and orientation as set forth in REP 103 [#REP103]_.
Discrepancies between a controller's internal Cartesian frame and REP 103 [#REP103]_ shall be resolved by defining the correct transform between ``base_link`` (or any other convenient frame) and the ``base`` frame (see `base`_).

``base_link`` may optionally have geometry (ie: 3D models) assigned to it.

No further special treatment of this frame is required.

Rationale: this is the standard ROS ``base_link`` frame, but used in the context of industrial robots.
As this frame is by convention considered the start of a ROS kinematic (sub) chain, using it in the same way in models of serial manipulators is strongly recommended, as this facilitates integration with other ROS tools and infrastructure.


base
''''

The ``base`` frame shall be coincident with the default Cartesian coordinate system as defined by the industrial robot controller.
Its purpose is to allow users to transform poses from a ROS application into the Cartesian base frame of the robot.
As such, this frame is exempt from the requirement to follow orientation conventions as described in REP 103 [#REP103]_.

Any frame is acceptable as the parent of ``base``, as long as the transform between parent and ``base`` is fixed (i.e.: not across a movable joint) and the location and orientation of the frame always correspond to the controller's internal default Cartesian frame.
As many serial manipulators share a common design, it is expected however that ``base_link`` will often be the parent of ``base``, connected by a ``fixed`` joint with a suitable transform.

``base`` shall not have any geometry associated with it.

Finally: ``base`` may be part of the (main) kinematic chain, but this is not required (ie: it could be a *leaf* frame).
Frame hierarchies must however support transforms from anywhere in the chain to ``base`` and vice-versa (for example: from ``base`` to ``tool0``, or from ``link_4`` to ``base``).

Rationale: for many industrial robots, the location of the ROS ``base_link`` frame is not necessarily coincident with the origin of their main Cartesian coordinate system.
This complicates expressing poses in robot-relative frames, as arbitrary transformations could be required for different makes and models of robots.
The ``base`` frame is intended to be used as an intermediary or bridge and provides a mapping that links the ROS TF tree with the controller- and vendor-specific coordinate systems and as an abstraction that decreases coupling between applications and specific robot urdfs.

Refer to `Vendor Nomenclature Mapping`_ for examples of vendor-specific names for this frame.


link_n
''''''

``link`` frames should be used for all link-local frames in the robot's kinematic chain.
As almost all manipulators are expected to require multiple such frames, each should be suffixed with a non-negative integer corresponding to the natural order of the frame in the chain from ``base_link`` to ``flange`` (ie: ``link_1``, ``link_2``, .., ``link_n; n ∈ ℕ``).

These frames shall follow ROS conventions for both chirality and orientation as set forth in REP 103 [#REP103]_.
If desirable, discrepancies between a controller's internal link-local frames and REP 103 [#REP103]_ may be resolved by defining a rigid transform between ``link_n`` and a suitably named proxy-frame (see `Robots with Left-handed Coordinate Systems`_ for instance).

If link-local frames are defined by the robot controller, authors should strive to make the location of ``link_n`` frames coincident with those frames, if they are externally accessible and/or usable for these purposes.

``link`` frames are expected to have robot geometry associated with them (as they provide a natural location for it), but this is not required.

Rationale: naming schemes for links and joints can vary between industrial robot manufacturers and even between robot series produced by the same manufacturer.
Allowing authors to import such implementation details into their robot models would immediately reduce reusability of both the models as well as of applications, as the latter would have to be changed to be compatible with the specific naming scheme being used (either through configuration or code adaptation).
Introducing a vendor-neutral naming scheme avoids this and also reduces the chances of misconfiguration.

In addition, harmonising link frame names across ROS supported (industrial) robots also allows users to make assumptions about such names and their semantics, facilitating development and workflows.

Finally: making ``link_n`` frames coincident with their counterparts on a robot controller allows such frames to be used as an intermediary or bridge and provides a mapping that links the ROS TF tree with controller- and vendor-specific coordinate systems.


flange
''''''

The ``flange`` frame is the frame that should be used to attach EEF models to the main kinematic chain of a ROS robot model.
In contrast to ``tool0``, this frame shall always be oriented such that it complies with REP 103 [#REP103]_.
Positive X (``x+``) must always point away from the last link (ie: in the 'forward' direction for a world-aligned robot model).

Any frame is acceptable as the parent of ``flange``, as long as the transform between that parent and ``flange`` is fixed (i.e.: not across a movable joint), it is located in the correct location and has the correct orientation.
It is expected that in most cases ``flange`` will be a child of the last physical link of a robot's kinematic chain (ie: the 6th or 7th link for a standard industrial serial manipulator).

As this frame is a virtual frame included here for the convenient attachment of EEF subassemblies to base robot models in ROS, there is no requirement for ``flange`` to be coincident with a *flange* frame if one is defined by the robot controller.
This REP does however recommend robot model authors to do so in those cases, as it is expected to reduce potential confusion and better match user expectations.

``flange`` shall not have any geometry associated with it.

Rationale: this separates the (physical) attachment point for EEFs from the mathematical TCP frame (which don't necessarily have to coincide for all robots, and also don't need to have the same orientation).
This makes attaching EEF models straightforward as no additional rotations are needed to align the EEF model with the robot flange link in a ROS model.


tool0
'''''

The ``tool0`` frame (pronounced: "tool-zero") shall match exactly an *all-zeros* TCP as defined by the robot controller.
As such, this frame is exempt from the requirement to follow orientation conventions as described in REP 103 [#REP103]_.
For most controllers, an all-zeros TCP is equal to an unconfigured (or default) TCP, which typically lies on the robot's physical mounting flange.
In this case the only expected difference between ``tool0`` and ``flange`` is the orientation.

Any frame is acceptable as the parent of ``tool0``, as long as the transform between that parent and ``tool0`` is fixed (i.e.: not across a movable joint), and ``tool0`` is located in the correct location and has the correct orientation.
It is however expected that in most cases ``tool0`` will be a child of the ``flange`` frame.
Whenever specific configurations require this other links may be used, but such deviations should be well justified and well documented (suitable candidates include the 6th or 7th link of industrial serial manipulators).

Neither the location nor the orientation of ``tool0`` in a robot model may be changed by users.
Instead, application-specific tool frames should be added as siblings of ``tool0`` (or could be defined in EEF sub-hierarchies) and should be named appropriately (see `Application-Specific Tool Frames`_).

``tool0`` shall not have any geometry associated with it.

Rationale: by not allowing changes to the location or orientation of ``tool0``, re-use of libraries such as kinematics solvers that are generated in an off-line fashion for a particular kinematic chain configuration becomes feasible.
It is the user's responsibility then to make sure that poses are transformed to the appropriate coordinate system before passing them on to such libraries (this could be done automatically by a motion planner or IK library based on configuration by the user, or manually before submitting goal poses to a planner).
Additionally: the purpose of ``tool0`` is to encode the location of an all-zeros or unconfigured tool frame.
Any changes to it would make it no longer match this default frame and would thereby defeat its purpose.

Refer to `Vendor Nomenclature Mapping`_ for examples of vendor-specific names for this frame.


Application-Specific Tool Frames
--------------------------------

It is strongly discouraged to use ``toolN`` names for application-specific tool frames, even if such a naming scheme is used by the robot controller(s) an application targets.
These names have very little semantic value, and the purpose of such TF frames cannot be properly understood without access to additional information external to a model itself.

Names with low semantic value are to be avoided in general, but in the case of robot tool frames this is especially important: use of an incorrect tool frame alone could lead to unexpected motion planning results which when executed could result in dangerous situations.

Users should therefore introduce additional frames to function as tool frames and give them appropriate names.
Any name is acceptable, as long as it is semantically meaningful and follows the naming guidelines for ROS resources as described in [#wiki_naming]_.

As explained in the `tool0`_ section, application-specific tool frames should be made siblings of the ``tool0`` frame and, as such, children of ``flange``.
Not using ``tool0`` as parent avoids introducing additional rotations (to resolve alignment issues due to ``tool0`` not adhering to REP 103 [#REP103]_) and facilitates reuse of frame data imported from robot controllers and external devices (such as tool frame calibration results, as such results are often relative to the robot's flange).

Finally: while this REP cannot prevent users from using names with low semantic value, ignoring this recommendation should be well justified and documented.


Dual-arm or Multi-group Robots
------------------------------

This REP does not specify any special conventions for robots with multiple arms, motion groups or kinematic chains.
The standard approach of prefixing joint and link names (with a `xacro` parameter for instance) to ensure uniqueness of all frames in the (combined) frame hierarchy should be used to avoid collisions.
See the `Example Frame Hierarchies`_ section for examples of this.


Robots with Left-handed Coordinate Systems
------------------------------------------

There are industrial manipulators that use a left-handed coordinate system for poses and in robot programming.
ROS exclusively uses a right-handed coordinate system, as described in REP 103 [#REP103]_.

As this fundamental difference cannot be resolved by using a transform, this REP recommends to overlay REP 103 [#REP103]_ compliant coordinate frames (ie: right-handed) and use conversion functions on the boundaries of ROS applications to convert data between such systems.


Frame Authorities
-----------------

The frames described in this REP will typically be part of the static description of robot models encoded in urdfs or xacros.
As such, the frame authority is expected to be an instance of ``robot_state_publisher``, but this is not required.
In cases where the kinematics of a particular robot model preclude the use of standard nodes, a specialised node capable of publishing the necessary frames could be used.


Exceptions
----------

The scope of potential robotics software is too broad to require all ROS software to follow the guidelines of this REP.
However, choosing different conventions should be well justified, well documented, and is discouraged.


Example Frame Hierarchies
=========================

This section shows a number of example frame hierarchies representative of typical kinematic configurations in industrial robotics and related contexts.
Each section includes an overview of the transform tree and a short description highlighting some noteworthy aspects.

Single manipulator
------------------

The following shows an example frame hierarchy for a single serial manipulator.
This particular example has ``base`` as a direct child of ``base_link``, the main kinematic chain starting with ``base_link`` and does not have any application-specific tool frame configured (ie: only has the default ``tool0`` frame)::

  base_link
  ├ base
  └ link_1
    └ ..
      └ link_N
        └ flange
          └ tool0

Note the absence of any prefixes: as there is only this single robot, they are not required.

Single manipulator with EEF
---------------------------

The following shows an example frame hierarchy for a single serial manipulator with an EEF model attached to ``flange`` and one application-specific tool frame (``eef_tcp``)::

  base_link
  ├ base
  └ link_1
    └ ..
      └ link_N
        └ flange
          ├ tool0
          ├ eef_base_link
          │   └ ..
          │     └ eef_link_N
          └ eef_tcp

Note the '``eef_``' prefix on the links in the EEF sub-hierarchy to prevent name clashes with the main robot model.

Note also that ``eef_tcp`` is a child of ``flange`` and not of ``eef_base_link``.
This is in accordance with `Application-Specific Tool Frames`_, as the EEF's TCP (in this example) is the result of a three-point calibration which was performed relative to the flange of the robot.

Multi-group (asymmetric)
------------------------

An example frame hierarchy for a setup that consists of two groups, a 6 axis industrial manipulator and a 2 axis positioner (or turntable).

Both are placed in the same work cell and share a common ``world`` frame::

  world
  ├ ..
  ├ robot_base_link
  │ ├ robot_base
  │ └ robot_link_1
  │   └ ..
  │     └ robot_link_N
  │       └ robot_flange
  │         └ robot_tool0
  └ positioner_base_link
    ├ positioner_base
    └ positioner_link_1
      └ positioner_link_2
        └ positioner_flange
          └ positioner_tool0

Note the '``robot_``' and '``positioner_``' prefixes on all frames.

Multi-group (symmetric)
-----------------------

The following shows an example frame hierarchy for a dual-arm robot that consists of two identical manipulators that are mirrored around a shared base.
Each arm sub-hierarchy has been given a prefix corresponding to its relative position::

  base_link
  ├ base
  ├ left_base_link
  │ ├ left_base
  │ └ left_link_1
  │   └ ..
  │     └ left_link_N
  │       └ left_flange
  │         └ left_tool0
  └ right_base_link
    ├ right_base
    └ right_link_1
      └ ..
        └ right_link_N
          └ right_flange
            └ right_tool0

Note that ``base_link`` in this example is the root of the entire robot structure and should be used when integrating the robot into a larger assembly.


Vendor Nomenclature Mapping
===========================

This section shows a mapping between vendor-specific frame nomenclature and the ``base`` and ``tool0`` frames as defined by this REP.

Note that for most vendors, ``tool0`` corresponds to an *all-zeros* tool frame configuration as described in the `tool0`_ section.
The names listed here in the *Vendor Name* column refer to the generic names for frames as used in the documentation of the control systems of the respective vendors, and not necessarily to any specific configurations of those frames.

+------------------+------------+---------------+
|                  | This REP   | Vendor Name   |
+==================+============+===============+
| ABB              | ``base``   | Base          |
| [#abb_opman]_    +------------+---------------+
|                  | ``tool0``  | TCP           |
+------------------+------------+---------------+
| Comau            | ``base``   | ``$BASE``     |
| [#comau_progg]_  +------------+---------------+
|                  | ``tool0``  | ``$TOOL``     |
+------------------+------------+---------------+
| Denso            | ``base``   | Base          |
| [#denso_pac]_    +------------+---------------+
|                  | ``tool0``  | Tool          |
+------------------+------------+---------------+
| Epson            | ``base``   | Robot         |
| [#epson_uguide]_ +------------+---------------+
|                  | ``tool0``  | TOOL 0        |
+------------------+------------+---------------+
| Fanuc            | ``base``   | WORLD         |
| [#fanuc_htool]_  +------------+---------------+
|                  | ``tool0``  | TOOL          |
+------------------+------------+---------------+
| Kawasaki         | ``base``   | Base          |
| [#kawa_opman]_   +------------+---------------+
|                  | ``tool0``  | Tool          |
+------------------+------------+---------------+
| KUKA             | ``base``   | ``$ROBROOT``  |
| [#kuka_kss]_     +------------+---------------+
|                  | ``tool0``  | ``$TOOL``     |
+------------------+------------+---------------+
| Mitsubishi       | ``base``   | Base          |
| [#mitsu_insman]_ +------------+---------------+
|                  | ``tool0``  | Tool          |
+------------------+------------+---------------+
| Staübli          | ``base``   | World         |
| [#staubli_val3]_ +------------+---------------+
|                  | ``tool0``  | tool          |
+------------------+------------+---------------+
| Universal        | ``base``   | Base          |
| Robots           +------------+---------------+
| [#ur_psman]_     | ``tool0``  | Tool          |
+------------------+------------+---------------+
| Yaskawa          | ``base``   | Robot         |
| Motoman          +------------+---------------+
| [#yask_fs100om]_ | ``tool0``  | Tool          |
+------------------+------------+---------------+


Compliance
==========

This REP depends on and is compliant with REP 103 [#REP103]_, except where stated otherwise.


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

.. [#wiki_naming] Names, ROS wiki, on-line, retrieved 24 April 2016
   (http://wiki.ros.org/Names)

.. [#abb_opman] ABB Robotics, Operating Manual, RobotStudio, 5.14, 3HAC032104-001, Revision F
.. [#comau_progg] Comau Robotics Instruction Handbook, C5G Controller Unit, MOTION PROGRAMMING, System Software Rel. 1.10, CR00757608_en-04/2011.07
.. [#denso_pac] DENSO Robot, PAC Programmer's Manual, Program Design and Commands
.. [#epson_uguide] EPSON, RC+ 5.0, User's Guide, Project Management and Development, Ver.5.4, EM135S2513F
.. [#fanuc_htool] FANUC Robot Series, R-30iA, Handling Tool, Operator's Manual
.. [#kawa_opman] Kawasaki Heavy Industries, Ltd., Kawasaki Robot Controller, E Series, Operation Manual, 90203-1104DED
.. [#kuka_kss] KUKA Roboter GmbH, KUKA System Software 8.3, Operating and Programming Instructions for System Integrators
.. [#mitsu_insman] MITSUBISHI, Mitsubishi Industrial Robot, CR750/CR751 Series Controller, INSTRUCTION MANUAL, BFP-A8869-D
.. [#staubli_val3] Stäubli, VAL3 Reference Manual
.. [#ur_psman] Universal Robots, Polyscope Manual, Version 3.9 (en)
.. [#yask_fs100om] Yaskawa, FS100 Operator's Manual, No. RE-CSO-A043


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
