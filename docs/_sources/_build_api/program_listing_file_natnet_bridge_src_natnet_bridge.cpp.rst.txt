
.. _program_listing_file_natnet_bridge_src_natnet_bridge.cpp:

Program Listing for File natnet_bridge.cpp
==========================================

|exhale_lsh| :ref:`Return to documentation for file <file_natnet_bridge_src_natnet_bridge.cpp>` (``natnet_bridge/src/natnet_bridge.cpp``)

.. |exhale_lsh| unicode:: U+021B0 .. UPWARDS ARROW WITH TIP LEFTWARDS

.. code-block:: cpp

   //
   // Created by yuan on 2021/5/23.
   //
   
   #include <natnet/natnet_definition.h>
   #include "natnet/socket.h"
   #include "natnet_bridge/natnet_bridge.h"
   #include "natnet/natnet_config.h"
   #include "natnet_bridge/server_parser.h"
   #include "natnet_bridge/unpack.h"
   
   #include "ros/ros.h"
   #include <cmath>
   
   namespace natnet_bridge{
   
       bool NatNetBridge::re_init_natnet(const NatNetBridgeDynParamConfig& config_in) {
           // FIXME change to from config
   
           data_port = atoi(config_in.data_port.c_str());
           command_port = atoi(config_in.command_port.c_str());
           multicast_ip_address = config_in.multicast_ip;
           ROS_INFO("multicast_ip_address %s Data Port %d,  Cammand Port %d",
                    multicast_ip_address.c_str(), data_port, command_port);
   
           multicastClientSocketPtr.reset(
                   new UdpMulticastSocket(data_port, multicast_ip_address)
           );
           return true;
       }
   
       bool NatNetBridge::re_init_publish(const NatNetBridgeDynParamConfig & config_in) {
           use_2d_bool = config_in.use_2d_bool;
           use_2d_plane_height = config_in.use_2d_plane_height;
   
           tf_world_frame = config_in.tf_world_frame;
           tf_optitrack_frame = config_in.tf_optitrack_frame;
   
           tf_filter_invalid_2d = config_in.tf_filter_invalid_2d;
           tf_filter_invalid_3d = config_in.tf_filter_invalid_3d;
   
           optitrack_coordinate = config_in.optitrack_coordinate;
   
           publish_unidentified = config_in.publish_unidentified;
           publish_skeleton = config_in.publish_skeleton;
           publish_rigid_body = config_in.publish_rigid_body;
           publish_marker_assets = config_in.publish_marker_assets;
           publish_labeled_marker = config_in.publish_labeled_marker;
           publish_unlabeled_marker = config_in.publish_unlabeled_marker;
   
           return true;
       }
   
       bool NatNetBridge::publish_rigid_2d_tf(const NatNetFrame & data_frame, ros::Time& time){
           //TODO, change it here
   
           if (this->optitrack_coordinate == NatNetBridgeDynParam_Z_UP){
               for (auto rb_i: data_frame.rigid_bodies) {
                   if (tf_filter_invalid_2d && rb_i.track_valid)
                       continue;
                   tf::Quaternion quad(rb_i.pose.orientation.x, rb_i.pose.orientation.y,
                                       rb_i.pose.orientation.z, rb_i.pose.orientation.w);
   
                   tf::Transform trans_i(
                       tf::createQuaternionFromRPY(0., 0., tf::getYaw(quad)),
                       tf::Vector3(rb_i.pose.position.x, rb_i.pose.position.y, this->use_2d_plane_height)
                   );
                   tfPublisher.sendTransform(
                       tf::StampedTransform(
                               trans_i, time,
                               this->tf_world_frame, "rigid_body_2d_"+std::to_string(rb_i.id))
                   );
               }
           } else if(this->optitrack_coordinate == NatNetBridgeDynParam_Y_UP){
               for (auto rb_i: data_frame.rigid_bodies) {
                   if (tf_filter_invalid_2d && rb_i.track_valid)
                       continue;
                   tf::Quaternion q_rot_W_Op, q_W_Rb, q_Op_Rb;
                   q_Op_Rb = tf::Quaternion(rb_i.pose.orientation.x, rb_i.pose.orientation.y,
                                            rb_i.pose.orientation.z, rb_i.pose.orientation.w);
   
                   q_rot_W_Op.setRPY(M_PI/2, 0., 0.);
                   q_W_Rb = q_rot_W_Op * q_Op_Rb;
                   q_W_Rb.normalize();
   
                   tf::Transform trans_i(
                           tf::createQuaternionFromRPY(0., 0., tf::getYaw(q_W_Rb)),
                           tf::Vector3(rb_i.pose.position.x, -rb_i.pose.position.z, this->use_2d_plane_height)
                   );
                   tfPublisher.sendTransform(
                           tf::StampedTransform(
                                   trans_i, time,
                                   this->tf_world_frame, "rigid_body_2d"+std::to_string(rb_i.id))
                   );
               }
           } else{
               ROS_ERROR("Not right optitrack coordinate definition");
           }
           return true;
       }
   
       bool NatNetBridge::publish_rigid_3d_tf(const NatNetFrame & data_frame, ros::Time& time){
           for (auto rigid_body_i : data_frame.rigid_bodies){
               if (tf_filter_invalid_3d && rigid_body_i.track_valid)
                   continue;
   
               tf::Vector3 r_i_point(
                       rigid_body_i.pose.position.x,rigid_body_i.pose.position.y,
                       rigid_body_i.pose.position.z);
   
               tf::Quaternion r_i_quaternion(
                       rigid_body_i.pose.orientation.x,rigid_body_i.pose.orientation.y,
                       rigid_body_i.pose.orientation.z, rigid_body_i.pose.orientation.w);
   
               tfPublisher.sendTransform(
                       tf::StampedTransform(
                               tf::Transform(r_i_quaternion, r_i_point),
                               time, this->tf_optitrack_frame,
                               "rigid_body" + std::to_string(rigid_body_i.id)));
           }
           return true;
       }
   
       bool NatNetBridge::receive_data() {
           natnet_bridge::NatNetFrame data_frame;
   
           int numBytesReceived = multicastClientSocketPtr->recv();
           if (numBytesReceived == -1){
               ROS_DEBUG("nothing received in this spin, use smaller loop rate for the receiving if this show frequently");
               return false;
           } else{
               const char* pMsgBuffer = multicastClientSocketPtr->getBuffer(); //TODO const?
               std::vector<char> msgBuffer(pMsgBuffer, pMsgBuffer + numBytesReceived);
   
               Unpack(msgBuffer.data(), natnet_version.v_major, natnet_version.v_minor, data_frame);
   
               ros::Time time = ros::Time::now();
   
               if (this->optitrack_coordinate == natnet_bridge::NatNetBridgeDynParam_Y_UP){
                   tf::Quaternion world2optitrack_quaternion(M_SQRT2/2., 0., 0., M_SQRT2/2.);
                   tf::Vector3 world2optitrack_vec(0., 0., 0.);
                   tfPublisher.sendTransform(
                       tf::StampedTransform(tf::Transform(world2optitrack_quaternion, world2optitrack_vec),
                                    time, this->tf_world_frame, tf_optitrack_frame));
               }
               else if (this->optitrack_coordinate == natnet_bridge::NatNetBridgeDynParam_Z_UP){
                   tf::Quaternion world2optitrack_quaternion(0., 0., 0., 0.);
                   tf::Vector3 world2optitrack_vec(0., 0., 0.);
                   tfPublisher.sendTransform(
                       tf::StampedTransform(tf::Transform(world2optitrack_quaternion, world2optitrack_vec),
                                   time, this->tf_world_frame, tf_optitrack_frame));
               }
               else
                   ROS_ERROR("Not right optitrack coordinate configuration");
   
               // Publish rigid bodies tf 2d
               this-> publish_rigid_3d_tf(data_frame, time);
               if (this->use_2d_bool)
                   this->publish_rigid_2d_tf(data_frame, time);
   
               if (publish_unidentified) data_frame.unlabeled_markers.clear();
               if (publish_skeleton) data_frame.skeletons.clear();
               if (publish_rigid_body) data_frame.rigid_bodies.clear();
               if (publish_marker_assets) data_frame.marker_sets.clear();
               if (publish_labeled_marker) data_frame.labeled_markers.clear();
               if (publish_unlabeled_marker) data_frame.unlabeled_markers.clear();
               data_frame.header.stamp = time;
               data_frame.header.frame_id = "header";
               data_frame.reference_frame = tf_optitrack_frame;
               natnet_frame_publisher.publish(data_frame);
               return true;
           }
       }
   
       bool NatNetBridge::connect_server() {
   
           bool connected= false;
           int n_trial = 1000;
   
           while( ros::ok() && (n_trial--)){
   
               std::vector<char> connect_command_msg;
               buildConnectPacket(connect_command_msg);
               int ret = multicastClientSocketPtr->send(connect_command_msg.data(), connect_command_msg.size(), command_port);
               if (ret == -1) ROS_ERROR("Failed for sending Msg CONNECT to server, I will try again ");
   
               int numBytesReceived = multicastClientSocketPtr->recv();
               if (numBytesReceived > 0){
                   // Grab latest message buffer
                   const char* pMsgBuffer = multicastClientSocketPtr->getBuffer();
                   std::vector<char> msgBuffer(pMsgBuffer, pMsgBuffer + numBytesReceived);
   
                   int success = server_info_parser(msgBuffer, this->server_version, this->natnet_version);
                   if (success){
                       ROS_INFO("Server Connected");
                       ROS_INFO("Server Version %s", server_version.getVersionString().c_str());
                       ROS_INFO("Natnet Version %s", natnet_version.getVersionString().c_str());
                       ROS_INFO("Server connected, will be able to receive data");
                       connected = true;
                       break;
                   }
               }
   
               ros::Duration(0.001).sleep();
               ros::spinOnce();
           }
   
           if (connected){
               return true;
           }
           else
               return false;
       }
   
       NatNetBridge::NatNetBridge(ros::NodeHandle& nh, const NatNetBridgeDynParamConfig& config): p_nh(nh){
           re_init_natnet(config);
           re_init_publish(config);
           natnet_frame_publisher = nh.advertise<natnet_bridge::NatNetFrame>("natnet_frame", 1000);
       }
   
   }
   
