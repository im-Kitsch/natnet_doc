
.. _program_listing_file_natnet_bridge_src_optitrack_node.cpp:

Program Listing for File optitrack_node.cpp
===========================================

|exhale_lsh| :ref:`Return to documentation for file <file_natnet_bridge_src_optitrack_node.cpp>` (``natnet_bridge/src/optitrack_node.cpp``)

.. |exhale_lsh| unicode:: U+021B0 .. UPWARDS ARROW WITH TIP LEFTWARDS

.. code-block:: cpp

   //
   // Created by yuan on 2021/5/22.
   //
   
   #include "ros/ros.h"
   #include "natnet/socket.h"
   #include "natnet_bridge/natnet_bridge.h"
   #include <dynamic_reconfigure/server.h>
   #include "natnet_bridge/NatNetBridgeDynParamConfig.h"
   #include <stdio.h>
   #include <natnet_bridge/NatNetFrame.h>
   #include <ros/console.h>
   
   class NatnetDynReconfigure{
   public:
       NatnetDynReconfigure(){
           publish_reconfigured = false;
           reset_reconfigured = false;
           dyn_config = natnet_bridge::NatNetBridgeDynParamConfig();
       }
       void dyn_reconfigure_cb(const natnet_bridge::NatNetBridgeDynParamConfig &config_in, uint32_t level){
           if (level & 0b10){
               ROS_INFO("Reconfigure Requested for Natnet Connection triggered");
               reset_reconfigured = true;
           }
           if (level & 0b01){
               ROS_INFO("Reconfigure Requested for publish triggered");
               publish_reconfigured = true;
           }
           dyn_config = config_in;
       }
       void reconfigure_responded(){
           publish_reconfigured = false;
           reset_reconfigured = false;
       };
   
   
       natnet_bridge::NatNetBridgeDynParamConfig dyn_config;
       bool reset_reconfigured;
       bool publish_reconfigured;
   };
   
   
   int main(int argc, char* argv[]){
   
       ros::init(argc, argv, "natnet_bridge_node");
       ros::NodeHandle nh("~");
   
       dynamic_reconfigure::Server<natnet_bridge::NatNetBridgeDynParamConfig> server;
       dynamic_reconfigure::Server<natnet_bridge::NatNetBridgeDynParamConfig>::CallbackType f;
       NatnetDynReconfigure natnet_reconfigure;
   
       f = boost::bind(&NatnetDynReconfigure::dyn_reconfigure_cb, &natnet_reconfigure, _1, _2);
       server.setCallback(f);
   
       natnet_bridge::NatNetBridge natnet_node(nh, natnet_reconfigure.dyn_config);
       natnet_reconfigure.reconfigure_responded();
   
       bool server_connected = natnet_node.connect_server();
   
       ros::Rate loop_rate(500);
       while (ros::ok())
       {
           // if dynamic_reconfigure triggered, re_configure node
           if (natnet_reconfigure.reset_reconfigured){
               natnet_node.re_init_natnet(natnet_reconfigure.dyn_config);
               server_connected = false;
               natnet_reconfigure.reset_reconfigured = false;
           }
           if (natnet_reconfigure.publish_reconfigured){
               natnet_node.re_init_publish(natnet_reconfigure.dyn_config);
               natnet_reconfigure.publish_reconfigured = false;
           }
   
           // if server not connected, connect again
           if (!server_connected){
               ROS_WARN("Server has not connected, trying to connect with server");
               server_connected = natnet_node.connect_server();
               loop_rate.reset(); // reset this since it costs much time
           }
   
           // receive data
           if (server_connected)
               natnet_node.receive_data();
           else{
               ROS_WARN("Received data is skipped due to server connection failed");
           }
   
           ros::spinOnce();
           bool not_over_loaded = loop_rate.sleep();
           if (!not_over_loaded)
               ROS_WARN("time overloaded for this spin");
       }
   
   }
