REP: XXX
Title: Layout of ROS Industrial manufacturer support repositories
Author: G.A. vd. Hoorn <g.a.vanderhoorn@tudelft.nl>
Status: Draft
Type: Process
Content-Type: text/x-rst
Created: 15-Nov-2013


Abstract
========

This document proposes a number of conventions for package naming
and layout within the context of robot manufacturer supporting
repositories in ROS Industrial. In addition, rules for naming
externally visible symbols and artefacts within those packages are
given.


Rationale
=========

text.


Definitions
===========

Manufacturer Support Repository
    The (version controlled) storage location which holds all
    packages belonging to a manufacturer metapackage.

Manufacturer Metapackage
    ROS-I metapackage grouping all driver, robot support and other
    packages for one particular manufacturer.

Driver Package
    ROS-I package containing all nodes required to implement support
    for the ROS-Industrial Driver Specification [#rosi_drv_spec]_.
    A manufacturer support repository may contain multiple driver
    packages, for instance when multiple incompatible controller
    models can be used with different robot models from the same
    manufacturer.

Robot Support Package
    ROS-I package containing all files needed to support a specific
    robot model (configuration files, meshes, urdfs and launch
    files). A single support package may support multiple robot
    models from the same family, by storing the union of all required
    files.

Robot Type
    Mostly used to describe its kinematic structure, applicability to
    certain use cases. Examples: SCARA or serial manipulator.

Robot Family (series)
    A set of manipulators all deriving from a common base model or
    set of functional characteristics.

Robot Model (variant?)
    One particular model of manipulator, member of a family, but
    completely defined by its model number. Examples are the Fanuc LR
    Mate 200iC/5H, the Motoman SIA50D, Adept Viper S650.


Names
=====

General
-------

All ROS-I packages shall follow normal ROS naming guidelines, as set
forth in [#resnames]_ and [#rosnames]_. Package names, file and
directory names as well as xacro macros follow these conventions.
In summary: start with a letter, no capitals, no special characters
and underscores in stead of spaces. This also holds for file
extensions: only lowercase is acceptable.

ROS-I specific examples: the Fanuc M-430iA/2F becomes ``m430ia2f``.
The Motoman HP20D/F becomes ``hp20df``.

Underscores should be used sparingly, but may be inserted to improve
readability of an identifier. Example: Adept Viper S650 could become
``viper_s650``, instead of ``vipers650``.


Manufacturer prefix
-------------------

text.


Single package per family
-------------------------

text.


Package Layout
==============

text.


Metapackage
-----------

This package shall be identical to all other ROS Catkin metapackages
[#catkinmeta]_. The package name should be equal to the name of the
support repository, but should always contain the name of the
manufacterer. All other packages in the stack should be listed as
``<run_depends>`` of the metapackage.

::

    MFG
     ├── CMakeLists.txt
     └── package.xml


Resource Package
----------------

Shall contain any artefacts (urdf, xacro, meshes, text files)
that are of use to other packages within the repository. This package
is a good place to store colour and material definitions common to
robot types within the repository, shared (workcell) meshes (such as
models of fixtures, turn tables, pedestals) and textures.

Meshes must be seperated into *collision* and *visual* grade variants.
Xacros and urdfs should reference the appropriate files in respective
``<collision>`` and ``<visual>`` tags. Acceptable file types for
meshes are those supported by RViz (ie: binary ``.stl``, ``.dae``
and Ogre ``.mesh``).

::

    MFG_resources
    ├── meshes
    │   ├── X
    │   │   ├── collision
    │   │   └── visual
    │   ├── Y
    │   │   └── ..
    │   └── ..
    ├── urdf
    │   ├── common_colours.xacro
    │   ├── common_materials.xacro
    │   └── ..
    ├── ..
    ├── CMakeLists.txt
    └── package.xml


Driver Package
--------------

This package shall contain nodes required for interfacing with
supported manipulators. If required, it may also contain code that is
to be run on the controller of the manipulator. In case the default
``industrial_robot_client`` nodes are insufficient, this package
should also contain specialised (subclassed) versions.

In addition, this package should contain the base launchfiles for
starting all ROS-I required nodes, with any required robot specific
parameters.

If sufficiently different, controller-specific prefixes could be
used. Example: ``motoman_fs100_driver``.

::

    MFG_driver
    ├── include
    │   └── ..
    ├── launch
    │   ├── motion_(streaming|download)_interface.launch
    │   ├── robot_interface_(streaming|download).launch
    │   ├── robot_state.launch
    │   └── ..
    ├── src
    │   └── ..
    ├── ..
    ├── CMakeLists.txt
    └── package.xml


Support Package
---------------

These packages add support for a family of manipulators. Support
packages shall contain configuration files, launchfiles, meshes
(collision and visual), urdfs and any other needed files --
for all supported models.

Additionally, the support package may provide functionality (in the
form of custom(ised) nodes or plugins) that is too specialised to be
included in the ``industrial_robot_client`` or any of the
repositories' driver packages. Examples could be filtering
or transform nodes that process incoming data from a controller
before passing it on to other ROS-I components (analogous to a
Windows filter driver).

In all cases, the external interface (topics and services) of
support packages which include such extensions must implement at least
the ROS-Industrial Driver Specification [#rosi_drv_spec]_.

::

    MFG_FAMILY_support
    ├── config
    │   ├── joint_names_MODEL_B1.yaml
    │   └── ..
    ├── launch
    │   ├── load_MODEL_B1.launch
    │   ├── robot_interface_(streaming|download)_MODEL_B1.launch
    │   ├── robot_state_visualize_MODEL_B1.launch
    │   ├── test_MODEL_B1.launch
    │   └── ..
    ├── meshes
    │   ├── MODEL_B1
    │   │   ├── collision
    │   │   │   ├── link_1.stl
    │   │   │   ├── link_2.stl
    │   │   │   └── ..
    │   │   └── visual
    │   │       ├── link_1.stl
    │   │       └── ..
    │   ├── MODEL_B2
    │   │   └── ..
    │   └─── ..
    ├── urdf
    │   ├── MODEL_B1.urdf
    │   ├── MODEL_B1.xacro
    │   ├── MODEL_B1_macro.xacro
    │   └── ..
    ├── CMakeLists.txt
    └── package.xml


Arm Navigation Package
----------------------

This package shall provide a ROS Arm Navigation compatible motion
planning interface for a specific robot model. Due to the nature of
these packages, only a single model is supported by each package.

NB: Arm Navigation is deprecated in ROS Groovy, and removed in ROS
Hydro. This section is included in this REP for retrospective
standardisation of existing packages.


MoveIt! Configuration Package
-----------------------------

Just keep normal MoveIt! setup assistant output, but follow ROS-I
naming conventions for package. As MoveIt! configurations are
specific to a robot model (and even per robot configuration), they
are separate packages, one per model.

**TODO**: should name of robot also be prefixed? In order to be able
to work with multiple moveit configs simultaneously?

::

    MFG_MODEL_moveit_config
    ├── config
    │   └── ..
    ├── launch
    │   └── ..
    ├── CMakeLists.txt
    └── package.xml

Note: any MoveIt! plugins should go in a separate package and should
not be integrated into the MoveIt! package itself.


MoveIt! Plugins Package
-----------------------

Package should contain all MoveIt! plugins (kinematics, controller
managers, etc) for a robot family. Should also be used to store
type specific MoveIt! filters.

Idea is similar to the single-support-package-per-family rule.

Inspired by the ``moveit_pr2`` package.

::

    MFG_FAMILY_moveit_plugins
    ├── MFG_MODEL_B1_kinematics
    │   ├── include
    │   ├── src
    │   ├── CMakeLists.txt
    │   ├── .._plugin_description.xml
    │   └── ..
    ├── MFG_MODEL_B2_kinematics
    │   ├── include
    │   ├── src
    │   ├── CMakeLists.txt
    │   ├── .._plugin_description.xml
    │   └── ..
    ├── MFG_MODEL_B2_filters
    │   ├── include
    │   ├── src
    │   ├── CMakeLists.txt
    │   ├── .._plugin_description.xml
    │   └── ..
    ├── CMakeLists.txt
    └── package.xml


Other Package
-------------

text.


Entity Naming
=============

Xacro Macros
------------

Follow other rules: ``mfg_prefix underscore model_name``.


URDF Robot Names
----------------

similar to Xacro macro.


References and Footnotes
========================

.. [#rosi_drv_spec] ROS-Industrial Robot Driver Specification (DRAFT). Date retrieved 2013-11-15. Online:
   http://wiki.ros.org/Industrial/Industrial_Robot_Driver_Spec

.. [#resnames] ROS Patterns - Conventions - Naming ROS Resources, ROS Wiki. Date retrieved 2013-11-15. Online:
   http://wiki.ros.org/ROS/Patterns/Conventions#Naming_ROS_Resources

.. [#rosnames] Names, ROS Wiki. Date retrieved 2013-11-15. Online:
   http://wiki.ros.org/Names

.. [#catkinmeta] Catkin - Metapackages, ROS Wiki. Date retrieved 2013-11-15. Online:
   http://wiki.ros.org/catkin/package.xml#Metapackages


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
