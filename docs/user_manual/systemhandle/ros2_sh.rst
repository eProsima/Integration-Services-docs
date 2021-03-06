.. _ros2_sh:

ROS 2 System Handle
===================

The *ROS 2 System Handle* can be used for two main purposes:

* Connection between a *ROS 2* application and an application running over a different middleware implementation.
  This is the classic use-case for *Integration Service*.

* Connecting two *ROS 2* applications running under different Domain IDs.

Dependencies
^^^^^^^^^^^^

The only dependency of this *System Handle* is to have a ROS 2 installation (`Foxy <https://docs.ros.org/en/foxy/Installation.html>`_ or superior) in your system.

Configuration
^^^^^^^^^^^^^

Regarding the *ROS 2 System Handle*, there are several specific parameters which can be configured
for the *ROS 2* middleware. All of these parameters are optional, and are suboptions of the main
five sections:

* :code:`systems`: The system :code:`type` must be :code:`ros2`. In addition to the
  :code:`type` and :code:`types-from` fields,
  the *ROS 2 System Handle* accepts the following specific configuration fields:

  .. code-block:: yaml

      systems:
        ros2:
          type: ros2
          namespace: "/"
          node_name: "my_ros2_node"
          domain: 4

  * :code:`namespace`: The *namespace* of the ROS 2 node created by the *ROS 2 System Handle*.

  * :code:`node_name`: The *ROS 2 System Handle* node name.

  * :code:`domain`: Provides with an easy way to change the *Domain ID* of the ROS 2 entities created
    by the *ROS 2 System Handle*.

* :code:`topics`: The topic :code:`route` must contain :code:`ros2` within its :code:`from` or :code:`to` fields. Additionally,
  the *ROS 2 System Handle* accepts the following topic specific configuration parameters, within the
  :code:`ros2` specific middleware configuration tag:

  .. code-block:: yaml

        routes:
          ros2_to_ros1: { from: ros2, to: ros1 }

        topics:
          hello_ros1:
            type: std_msgs/String
            route: ros2_to_ros1
            ros2: { qos: {
                deadline: { sec: 1, nanosec: 10},
                durability: VOLATILE,
                history: { kind: KEEP_LAST, depth: 10 },
                lifespan: { sec: 2, nanosec: 20 },
                liveliness: { kind: AUTOMATIC, sec: 2, nanosec: 0 },
                reliability: RELIABLE
              }}

  * :code:`qos`: The Quality of Service policies that are going to be applied to the ROS 2 entity involved in the pub-sub operation (in this case the publisher).
    This parameter accepts any of the QoS available for ROS 2:

    * :code:`deadline`: This QoS policy raises an alarm when the frequency of new samples falls below a certain threshold, which can be defined by means of the
      :code:`sec` and :code:`nanosec` tags. On the publishing side, the deadline defines the maximum period in which the application is expected to supply a new
      sample. On the subscribing side, it defines the maximum period in which new samples should be received.

    * :code:`durability`: A Publisher can send messages throughout a Topic even if there are no Subscribers on the network. This QoS defines how the system will
      behave regarding those samples that existed on the Topic before the Subscriber joins. There are two possible values: :code:`VOLATILE` and
      :code:`TRANSIENT_LOCAL`.

    * :code:`history`: This QoS controls the behavior of the system when the value of an instance changes one or more times before it can be successfully
      communicated to the existing Subscriber entities.

      * :code:`kind`: Controls if the service should deliver only the most recent values, all the intermediate values or do something in between. There are two
        possible values: :code:`KEEP_LAST` and :code:`KEEP_ALL`.

      * :code:`depth`: Establishes the maximum number of samples that must be kept on the history. It only has effect if the kind is set to :code:`KEEP_LAST`.

    * :code:`lifespan`: Each data sample written by a Publisher has an associated expiration time beyond which the data is removed from the Publisher and
      Subscriber history. That expiration time can be defined by means of the :code:`sec` and :code:`nanosec` tags.

    * :code:`liveliness`: This QoS controls the mechanism used by the service to ensure that a particular entity on the network is still alive.

      * :code:`kind`: Establishes if the service needs to assert the liveliness automatically or if it needs to wait until the liveliness is asserted by the
        publishing side. There are two possible values: :code:`AUTOMATIC` and :code:`MANUAL_BY_TOPIC`.

      * :code:`lease_duration`: Amount of time to wait since the last time the Publisher asserts its liveliness to consider that it is no longer alive. It can be
        defined by means of the :code:`sec` and :code:`nanosec` tags.

    * :code:`reliability`: This QoS indicates the level of reliability offered and requested by the service.
      There are two possible values: :code:`RELIABLE` and :code:`BEST_EFFORT`.

Examples
^^^^^^^^

There are several examples that you can find in this documentation in which the
*ROS 2 System Handle* is employed in the communication process. Some of them are presented here:

* :ref:`ros1_ros2_bridge_pubsub`
* :ref:`dds_ros2_bridge_pubsub`
* :ref:`ros2_websocket_bridge_pubsub`
* :ref:`fiware_ros2_bridge_pubsub`
* :ref:`ros2_server_bridge`
* :ref:`ros2_change_of_domain`

.. _ros2_compilation_flags:

Compilation flags
^^^^^^^^^^^^^^^^^


Besides the :ref:`global_compilation_flags` available for the
whole *Integration Service* product suite, there are some specific flags which apply only to the
*ROS 2 System Handle*; they are listed below:

* :code:`BUILD_ROS2_TESTS`: Allows to specifically compile the *ROS 2 System Handle* unitary and
  integration tests. It is useful to avoid compiling each *System Handle*'section test suite present
  in the :code:`colcon` workspace, which is what would happen if using the :code:`BUILD_TESTS` flag,
  with the objective of minimizing building time. To use it, after making sure that the *ROS 2 System Handle*
  is present in your :code:`colcon` workspace, execute the following command:

  .. code-block:: bash

      ~/is_ws$ colcon build --cmake-args -DBUILD_ROS2_TESTS=ON

* :code:`IS_ROS2_DISTRO`: This flag is intended to select the *ROS 2* distro that should be used to compile
  the *ROS 2 System Handle*. If not set, the version will be retrieved from the last *ROS distro* sourced in
  the compilation environment; this means that if the last *ROS* environment sourced corresponds to *ROS 1*,
  the compilation process will stop and warn the user about it.

* :code:`MIX_ROS_PACKAGES`: It accepts as an argument a list of `ROS packages <https://index.ros.org/packages/>`_,
  such as :code:`std_msgs`, :code:`geometry_msgs`, :code:`sensor_msgs`, :code:`nav_msgs`...
  for which the required transformation library to convert the specific *ROS 2* type definitions into *xTypes*,
  and the other way around, will be built. This list is shared with the `ROS 1 System Handle <https://github.com/eProsima/ROS1-SH#compilation-flags>`_,
  meaning that the ROS packages specified in the `MIX_ROS_PACKAGES` variable will also be built for *ROS 1*
  if the corresponding *System Handle* is present within the *Integration Service* workspace.
  To avoid possible errors, if a certain package is only present in *ROS 2*,
  the `MIX_ROS2_PACKAGES` flag must be used instead.

  These transformation libraries are also known within the *Integration Service*
  context as :code:`Middleware Interface Extension` or :code:`mix` libraries.

  By default, only the :code:`std_msgs_mix` library is compiled, unless the :code:`BUILD_TESTS`
  or :code:`BUILD_ROS2_TESTS` is used, case in which some additional ROS 2 packages :code:`mix` files
  required for testing will be built.

  If the user wants to compile some additional packages to use them with *Integration Service*,
  the following command must be launched to compile it, adding as much packages to the list as desired:

  .. code-block:: bash

      ~/is_ws$ colcon build --cmake-args -DMIX_ROS_PACKAGES="std_msgs geometry_msgs sensor_msgs nav_msgs"

* :code:`MIX_ROS2_PACKAGES`: It is used just as the `MIX_ROS_PACKAGES` flag, but will only affect *ROS 2*;
  this means that the `mix` generation engine will not search within the *ROS 1* packages,
  allowing to compile specific *ROS 2* packages independently.

  For example, if a user wants to compile a certain package `dummy_msgs` independently from *ROS 2*,
  but compiling `std_msgs` and `geometry_msgs` for both the *ROS 1* and *ROS 2 System Handles*,
  the following command should be executed:

  .. code-block:: bash

      ~/is_ws$ colcon build --cmake-args -DMIX_ROS_PACKAGES="std_msgs geometry_msgs" -DMIX_ROS2_PACKAGES="dummy_msgs"


API Reference
^^^^^^^^^^^^^

The *Integration Service API Reference* constitutes an independent section within this documentation.
To access the *ROS 2 System Handle* subsection, use this :ref:`link <api_is_ros2_sh>`.
