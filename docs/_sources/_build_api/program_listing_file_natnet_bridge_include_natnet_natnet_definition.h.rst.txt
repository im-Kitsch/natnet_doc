
.. _program_listing_file_natnet_bridge_include_natnet_natnet_definition.h:

Program Listing for File natnet_definition.h
============================================

|exhale_lsh| :ref:`Return to documentation for file <file_natnet_bridge_include_natnet_natnet_definition.h>` (``natnet_bridge/include/natnet/natnet_definition.h``)

.. |exhale_lsh| unicode:: U+021B0 .. UPWARDS ARROW WITH TIP LEFTWARDS

.. code-block:: cpp

   //
   // Created by yuan on 2021/5/23.
   // Copied from NetNet API of motive
   //
   
   #ifndef NATNET_BRIDGE_NATNET_DEFINITION_H
   #define NATNET_BRIDGE_NATNET_DEFINITION_H
   
   #include <vector>
   
   
   #define MAX_NAMELENGTH              256
   #define MAX_PACKETSIZE              100000  // max size of packet (actual packet size is dynamic)
   
   namespace natnet_bridge
   {
       typedef std::vector<char> MessageBuffer;
   
       // sSender and sPacket, copied from official API
       // sender
       typedef struct
       {
           char szName[MAX_NAMELENGTH];            // sending app's name
           unsigned char Version[4];               // sending app's version [major.minor.build.revision]
           unsigned char NatNetVersion[4];         // sending app's NatNet version [major.minor.build.revision]
       } __attribute__ ((__packed__)) sSender;
   
       typedef struct
       {
           unsigned short iMessage;                // message ID (e.g. NAT_FRAMEOFDATA)
           unsigned short nDataBytes;              // Num bytes in payload
           union
           {
               unsigned char  cData[MAX_PACKETSIZE];
               char           szData[MAX_PACKETSIZE];
               unsigned long  lData[MAX_PACKETSIZE/4];
               float          fData[MAX_PACKETSIZE/4];
               sSender        Sender;
           } Data;                                 // Payload incoming from NatNet Server
   
       }  __attribute__ ((__packed__)) sPacket;
   }
   
   #endif //NATNET_BRIDGE_NATNET_DEFINITION_H
