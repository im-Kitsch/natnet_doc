
.. _program_listing_file_natnet_bridge_include_natnet_natnet_config.h:

Program Listing for File natnet_config.h
========================================

|exhale_lsh| :ref:`Return to documentation for file <file_natnet_bridge_include_natnet_natnet_config.h>` (``natnet_bridge/include/natnet/natnet_config.h``)

.. |exhale_lsh| unicode:: U+021B0 .. UPWARDS ARROW WITH TIP LEFTWARDS

.. code-block:: cpp

   //
   // Created by yuan on 2021/5/24.
   //
   
   #ifndef NATNET_BRIDGE_NATNET_CONFIG_H
   #define NATNET_BRIDGE_NATNET_CONFIG_H
   
   #include <string>
   
   namespace natnet_config{
   
       namespace MessageType {
           const int Connect                   = 0;
           const int ServerInfo                = 1;
           const int Request                   = 2;
           const int Response                  = 3;
           const int RequestModelDef           = 4;
           const int ModelDef                  = 5;
           const int RequestFrameOfData        = 6;
           const int FrameOfData               = 7;
           const int MessageString             = 8;
           const int UnrecognizedRequest       =100;
           const int Undefined                 =999999;
       }
   
   
   //    namespace ros_param
   //    {   //TODO, static const?
   //        const std::string MulticastIpAddress_key = "optitrack_config/multicast_address";
   //        const std::string MulticastIpAddress_default = "239.255.42.99";   // TODO, use DEFINE
   //        const std::string CommandPort_key = "optitrack_config/command_port";
   //        const int         CommandPort_default = 1510;
   //        const std::string DataPort_key = "optitrack_config/data_port";
   //        const int         DataPort_default = 1511;
   //        //TODO, enable needed?
   //    }  // namespace ros_param
   
   }
   
   #endif //NATNET_BRIDGE_NATNET_CONFIG_H
