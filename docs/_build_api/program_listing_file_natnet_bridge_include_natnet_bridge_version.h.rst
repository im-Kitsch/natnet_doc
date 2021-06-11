
.. _program_listing_file_natnet_bridge_include_natnet_bridge_version.h:

Program Listing for File version.h
==================================

|exhale_lsh| :ref:`Return to documentation for file <file_natnet_bridge_include_natnet_bridge_version.h>` (``natnet_bridge/include/natnet_bridge/version.h``)

.. |exhale_lsh| unicode:: U+021B0 .. UPWARDS ARROW WITH TIP LEFTWARDS

.. code-block:: cpp

   //
   // Created by yuan on 2021/5/23.
   //
   
   #ifndef NATNET_BRIDGE_VERSION_H
   #define NATNET_BRIDGE_VERSION_H
   #include <string>
   
   // FIXME move it to natnet
   
   namespace natnet_bridge
   {
   
       class Version
       {
       public:
           Version();
           Version(int major, int minor, int revision, int build);
           Version(const std::string& version);
           ~Version();
   
           void setVersion(int major, int minor, int revision, int build);
           std::string const& getVersionString() const;
           bool operator > (const Version& comparison) const;
           bool operator >= (const Version& comparison) const;
           bool operator < (const Version& comparison) const;
           bool operator <= (const Version& comparison) const;
           bool operator == (const Version& comparison) const;
   
           int v_major;
           int v_minor;
           int v_revision;
           int v_build;
           std::string v_string;
       };
   
   }
   
   #endif //NATNET_BRIDGE_VERSION_H
