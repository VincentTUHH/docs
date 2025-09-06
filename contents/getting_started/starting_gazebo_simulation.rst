Starting Simulation
###################


General Launch Setup
====================

Vehicle-specific nodes should be launched in namespace "vehicle_name", for example :code:`uuv00` - :code:`uuv99`.

Each camera node should be launched in namespace "camera_name", which can be: :code:`vertical_camera` for HippoCampus, and :code:`vertical_camera` and :code:`front_camera` for the BlueROV's two cameras.


Spawning HippoCampus
====================

To start Gazebo with one HippoCampus:

.. code-block:: console

   $ ros2 launch hippo_sim top_hippocampus_complete.launch.py vehicle_name:=uuv00 start_gui:=true

As an example, we use :code:`uuv00` as the vehicle name. Per default, we do not open the Gazebo GUI but use RVIZ instead for visualization. If you want to see the Gazebo GUI, you need to set the respective launch argument to true: :code:`start_gui:=true`.

This launchfile starts the simualtion and spawns the vehicle, the tank and, optionally, AprilTags on the floor. 
It also starts some utilities such as the camera bridge between Gazebo and ROS2, our tf helper for basic transformations, as well as the vehicle-specific actuator mixer. 


Path Following example
======================

As an example, we will let the HippoCampus follow an Infinity-shaped path in Gazebo.

.. code-block:: console

   $ ros2 launch hippo_control top_path_following_intra_process.launch.py vehicle_name:=uuv00 use_sim_time:=true

This will start the path follower node and all other necessary controllers.

However, by default, the thrust setpoint will be zero. To move forward, set the thrust to a positive value by publishing to the respective topic:

.. code-block:: console

   $ ros2 topic pub -r 50 /uuv00/thrust_setpoint hippo_control_msgs/msg/ActuatorSetpoint "{x: 0.8}"


.. figure:: /res/images/hippo_inf_path_in_simulation.gif
   :align: center
   :width: 500

   HippoCampus in Gazebo


