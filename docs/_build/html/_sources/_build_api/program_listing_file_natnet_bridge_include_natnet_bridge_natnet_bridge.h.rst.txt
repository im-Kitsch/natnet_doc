
.. _program_listing_file_natnet_bridge_include_natnet_bridge_natnet_bridge.h:

Program Listing for File natnet_bridge.h
========================================

|exhale_lsh| :ref:`Return to documentation for file <file_natnet_bridge_include_natnet_bridge_natnet_bridge.h>` (``natnet_bridge/include/natnet_bridge/natnet_bridge.h``)

.. |exhale_lsh| unicode:: U+021B0 .. UPWARDS ARROW WITH TIP LEFTWARDS

.. code-block:: cpp

   //
   // Created by yuan on 2021/5/23.
   //
   
   #ifndef NATNET_BRIDGE_NATNET_BRIDGE_H
   #define NATNET_BRIDGE_NATNET_BRIDGE_H
   #include <ros/ros.h>
   #include "natnet_bridge/version.h"
   #include "tf/transform_broadcaster.h"
   #include "natnet_bridge/NatNetBridgeDynParamConfig.h"
   #include "natnet_bridge/NatNetFrame.h"
   #include <cstdlib>
   
   namespace natnet_bridge{
   
   class NatNetBridge{
   public:
       NatNetBridge(ros::NodeHandle& nh, const NatNetBridgeDynParamConfig& config);  // explicit according to warning
       bool re_init_natnet(const NatNetBridgeDynParamConfig&);
       bool re_init_publish(const NatNetBridgeDynParamConfig&);
       bool connect_server();
       bool receive_data();
   
   
   private:
       bool publish_rigid_2d_tf(const NatNetFrame & data_frame, ros::Time& time);
       bool publish_rigid_3d_tf(const NatNetFrame & data_frame, ros::Time& time);
   
       ros::NodeHandle p_nh;
       Version server_version;
       Version natnet_version;
   
       // publish config for 2d publish
       bool use_2d_bool;
       double use_2d_plane_height;
       bool tf_filter_invalid_2d, tf_filter_invalid_3d;
       // publish config for dataframe publish
       bool publish_unidentified, publish_skeleton, publish_rigid_body;
       bool publish_marker_assets, publish_labeled_marker, publish_unlabeled_marker;
       // tf tree settings
       std::string tf_world_frame;
       std::string tf_optitrack_frame;
       int optitrack_coordinate;
   
       std::string multicast_ip_address;
       int command_port;
       int data_port;
   
       std::unique_ptr<UdpMulticastSocket> multicastClientSocketPtr;
   
       ros::Publisher natnet_frame_publisher;
       tf::TransformBroadcaster tfPublisher;
   };
   
   } // end of natnet_bridge
   #endif //NATNET_BRIDGE_NATNET_BRIDGE_H
