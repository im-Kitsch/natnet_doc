
.. _program_listing_file_natnet_bridge_include_natnet_bridge_unpack.h:

Program Listing for File unpack.h
=================================

|exhale_lsh| :ref:`Return to documentation for file <file_natnet_bridge_include_natnet_bridge_unpack.h>` (``natnet_bridge/include/natnet_bridge/unpack.h``)

.. |exhale_lsh| unicode:: U+021B0 .. UPWARDS ARROW WITH TIP LEFTWARDS

.. code-block:: cpp

   //
   // Created by yuan on 2021/5/30.
   //
   
   #ifndef NATNET_BRIDGE_UNPACK_H
   #define NATNET_BRIDGE_UNPACK_H
   #include "natnet_bridge/NatNetFrame.h"
   
   
   namespace natnet_bridge{
   
   void Unpack(char* pData, int major, int minor, natnet_bridge::NatNetFrame& natnet_data);
   
   }
   
   #endif //NATNET_BRIDGE_UNPACK_H
