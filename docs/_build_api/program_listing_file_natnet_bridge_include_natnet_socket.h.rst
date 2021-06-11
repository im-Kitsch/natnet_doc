
.. _program_listing_file_natnet_bridge_include_natnet_socket.h:

Program Listing for File socket.h
=================================

|exhale_lsh| :ref:`Return to documentation for file <file_natnet_bridge_include_natnet_socket.h>` (``natnet_bridge/include/natnet/socket.h``)

.. |exhale_lsh| unicode:: U+021B0 .. UPWARDS ARROW WITH TIP LEFTWARDS

.. code-block:: cpp

   //
   // Created by yuan on 2021/5/23.
   //
   
   #ifndef NATNET_BRIDGE_SOCKET_H
   #define NATNET_BRIDGE_SOCKET_H
   
   #include <sys/types.h>
   #include <sys/socket.h>
   #include <netinet/in.h>
   #include <netdb.h>
   #include <unistd.h>
   #include <string>
   #include <arpa/inet.h>
   #include <stdexcept>
   
   //FIXME move to natnet
   
   class SocketException : public std::runtime_error
   {
   public:
   
       SocketException(std::string description) : std::runtime_error(description) {}
   
       ~SocketException() throw() {}
   };
   
   class UdpMulticastSocket
   {
   public:
   
       static const int MAXRECV = 3000;
   
       UdpMulticastSocket(const int local_port, const std::string multicast_ip = "224.0.0.1");
   
       ~UdpMulticastSocket();
   
       int recv();
   
       const char* getBuffer()
       {
           return &buf[0];
       }
   
       int send(const char* buf, unsigned int sz, int port);
   
   private:
   
       int m_socket;
       sockaddr_in m_local_addr;
       sockaddr_in HostAddr;
       bool remote_ip_exist;
       char buf [ MAXRECV + 1 ];
   };
   
   #endif //NATNET_BRIDGE_SOCKET_H
