
.. _program_listing_file_natnet_bridge_src_unpack.cpp:

Program Listing for File unpack.cpp
===================================

|exhale_lsh| :ref:`Return to documentation for file <file_natnet_bridge_src_unpack.cpp>` (``natnet_bridge/src/unpack.cpp``)

.. |exhale_lsh| unicode:: U+021B0 .. UPWARDS ARROW WITH TIP LEFTWARDS

.. code-block:: cpp

   //
   // Created by yuan on 2021/5/29.
   //
   #include <cstring>
   #include <string>
   #include <inttypes.h>
   #include "ros/ros.h"
   
   #include "natnet_bridge/unpack.h"
   
   
   #define MAX_NAMELENGTH              256
   
   // TODO, ROS_DEBUG as verbose
   
   namespace natnet_bridge{
   
   // non-standard/optional extension of C; define an unsafe version here
   // to not change example code below
   int strcpy_s(char* dest, const char *src)
   {
       strcpy(dest, src);
       return 0;
   }
   
   template<typename... Args> int sprintf_s(char * buffer, size_t bufsz, const char * format, Args... args)
   {
       return sprintf(buffer, format, args...);
   }
   
   
   // Funtion that assigns a time code values to 5 variables passed as arguments
   // Requires an integer from the packet as the timecode and timecodeSubframe
   bool DecodeTimecode(unsigned int inTimecode, unsigned int inTimecodeSubframe, int* hour, int* minute, int* second, int* frame, int* subframe)
   {
       bool bValid = true;
   
       *hour = (inTimecode>>24)&255;
       *minute = (inTimecode>>16)&255;
       *second = (inTimecode>>8)&255;
       *frame = inTimecode&255;
       *subframe = inTimecodeSubframe;
   
       return bValid;
   }
   
   // Takes timecode and assigns it to a string
   bool TimecodeStringify(unsigned int inTimecode, unsigned int inTimecodeSubframe, char *Buffer, int BufferSize)
   {
       bool bValid;
       int hour, minute, second, frame, subframe;
       bValid = DecodeTimecode(inTimecode, inTimecodeSubframe, &hour, &minute, &second, &frame, &subframe);
   
       sprintf_s(Buffer,BufferSize,"%2d:%2d:%2d:%2d.%d",hour, minute, second, frame, subframe);
       for(unsigned int i=0; i<strlen(Buffer); i++)
           if(Buffer[i]==' ')
               Buffer[i]='0';
   
       return bValid;
   }
   
   void DecodeMarkerID(int sourceID, int* pOutEntityID, int* pOutMemberID)
   {
       if (pOutEntityID)
           *pOutEntityID = sourceID >> 16;
   
       if (pOutMemberID)
           *pOutMemberID = sourceID & 0x0000ffff;
   }
   
   // *********************************************************************
   //
   //  Unpack Data:
   //      Recieves pointer to bytes that represent a packet of data
   //
   //      There are lots of print statements that show what
   //      data is being stored
   //
   //      Most memcpy functions will assign the data to a variable.
   //      Use this variable at your descretion.
   //      Variables created for storing data do not exceed the
   //      scope of this function.
   //
   // *********************************************************************
   void Unpack(char* pData, int major, int minor, natnet_bridge::NatNetFrame& natnet_data)
   {
       // Checks for NatNet Version number. Used later in function. Packets may be different depending on NatNet version.
   //    int major = NatNetVersion[0];
   //    int minor = NatNetVersion[1];
   
       char *ptr = pData;
   
   //    printf("Begin Packet\n-------\n");
       ROS_DEBUG("--------- Begin Packet -------");
   
       // First 2 Bytes is message ID
       int MessageID = 0;
       memcpy(&MessageID, ptr, 2); ptr += 2;
   //    printf("Message ID : %d\n", MessageID);
       ROS_DEBUG("Message ID : %d", MessageID);
   
       // Second 2 Bytes is the size of the packet
       int nBytes = 0;
       memcpy(&nBytes, ptr, 2); ptr += 2;
   //    printf("Byte count : %d\n", nBytes);
       ROS_DEBUG("Byte count : %d", nBytes);
   
       if(MessageID == 7)      // FRAME OF MOCAP DATA packet
       {
           // Next 4 Bytes is the frame number
           int frameNumber = 0; memcpy(&frameNumber, ptr, 4); ptr += 4;
   //        printf("Frame # : %d\n", frameNumber);
           ROS_DEBUG("Frame # : %d", frameNumber);
           natnet_data.natnet_frame_number = frameNumber;
   
           // Next 4 Bytes is the number of data sets (markersets, rigidbodies, etc)
           int nMarkerSets = 0; memcpy(&nMarkerSets, ptr, 4); ptr += 4;
   //        printf("Marker Set Count : %d\n", nMarkerSets);
           ROS_DEBUG("Marker Set Count : %d", nMarkerSets);
   
           // Loop through number of marker sets and get name and data
           for (int i=0; i < nMarkerSets; i++)
           {
               // Markerset name
               char szName[256];
               strcpy_s(szName, ptr);
               int nDataBytes = (int) strlen(szName) + 1;
               ptr += nDataBytes;
   //            printf("Model Name: %s\n", szName);
               ROS_DEBUG("Model Name: %s", szName);
   
               // marker data
               int nMarkers = 0; memcpy(&nMarkers, ptr, 4); ptr += 4;
   //            printf("Marker Count : %d\n", nMarkers);
               ROS_DEBUG("Marker Count : %d", nMarkers);
   
               natnet_bridge::MarkerSet marker_set;
               marker_set.name = szName;
               marker_set.n_markers = nMarkers;
   
               for(int j=0; j < nMarkers; j++)
               {
                   float x = 0; memcpy(&x, ptr, 4); ptr += 4;
                   float y = 0; memcpy(&y, ptr, 4); ptr += 4;
                   float z = 0; memcpy(&z, ptr, 4); ptr += 4;
   //                printf("\tMarker %d : [x=%3.2f,y=%3.2f,z=%3.2f]\n",j,x,y,z);
                   ROS_DEBUG("\tMarker %d : [x=%3.2f,y=%3.2f,z=%3.2f] ",j,x,y,z);
   //                marker_set.markers.push_back(natnet_bridge::PointFloat32(x, y, z));
                   natnet_bridge::PointFloat32 pp;
                   pp.x = x; pp.y = y;  pp.z = z;
                   marker_set.markers.push_back(pp);
               }
               natnet_data.marker_sets.push_back(marker_set);
           }
   
           // Loop through unidentified markers
           int nOtherMarkers = 0; memcpy(&nOtherMarkers, ptr, 4); ptr += 4;
           // OtherMarker list is Deprecated
           //printf("Unidentified Marker Count : %d\n", nOtherMarkers);
           ROS_DEBUG("Unidentified Marker Count : %d ", nOtherMarkers);
           for(int j=0; j < nOtherMarkers; j++)
           {
               float x = 0.0f; memcpy(&x, ptr, 4); ptr += 4;
               float y = 0.0f; memcpy(&y, ptr, 4); ptr += 4;
               float z = 0.0f; memcpy(&z, ptr, 4); ptr += 4;
   
               // Deprecated
               //printf("\tMarker %d : pos = [%3.2f,%3.2f,%3.2f]\n",j,x,y,z);
               ROS_DEBUG("\tMarker %d : pos = [%3.2f,%3.2f,%3.2f]",j,x,y,z);
               natnet_bridge::PointFloat32 marker;
               marker.x = x; marker.y = y; marker.z = z;
               natnet_data.unidentified_markers.push_back(marker);
           }
   
           // Loop through rigidbodies
           int nRigidBodies = 0;
           memcpy(&nRigidBodies, ptr, 4); ptr += 4;
   //        printf("Rigid Body Count : %d\n", nRigidBodies);
           ROS_DEBUG("Rigid Body Count : %d", nRigidBodies);
           for (int j=0; j < nRigidBodies; j++)
           {
               // Rigid body position and orientation
               int ID = 0; memcpy(&ID, ptr, 4); ptr += 4;
               float x = 0.0f; memcpy(&x, ptr, 4); ptr += 4;
               float y = 0.0f; memcpy(&y, ptr, 4); ptr += 4;
               float z = 0.0f; memcpy(&z, ptr, 4); ptr += 4;
               float qx = 0; memcpy(&qx, ptr, 4); ptr += 4;
               float qy = 0; memcpy(&qy, ptr, 4); ptr += 4;
               float qz = 0; memcpy(&qz, ptr, 4); ptr += 4;
               float qw = 0; memcpy(&qw, ptr, 4); ptr += 4;
   //            printf("ID : %d\n", ID);
   //            printf("pos: [%3.2f,%3.2f,%3.2f]\n", x,y,z);
   //            printf("ori: [%3.2f,%3.2f,%3.2f,%3.2f]\n", qx,qy,qz,qw);
               ROS_DEBUG("ID : %d\n", ID);
               ROS_DEBUG("pos: [%3.2f,%3.2f,%3.2f]\n", x,y,z);
               ROS_DEBUG("ori: [%3.2f,%3.2f,%3.2f,%3.2f]\n", qx,qy,qz,qw);
               natnet_bridge::RigidBody rigid_body;
               rigid_body.id = ID;
               rigid_body.pose.position.x = x; rigid_body.pose.position.y = y; rigid_body.pose.position.z = z;
               rigid_body.pose.orientation.w = qw; rigid_body.pose.orientation.x = qx;
               rigid_body.pose.orientation.y = qy; rigid_body.pose.orientation.z = qz;
   
   
               // NatNet version 2.0 and later
               if(major >= 2)
               {
                   // Mean marker error
                   float fError = 0.0f; memcpy(&fError, ptr, 4); ptr += 4;
   //                printf("Mean marker error: %3.2f\n", fError);
   
                   ROS_DEBUG("Mean marker error: %3.2f\n", fError);
                   rigid_body.error = fError;
               }
   
               // NatNet version 2.6 and later
               if( ((major == 2)&&(minor >= 6)) || (major > 2) || (major == 0) )
               {
                   // params
                   short params = 0; memcpy(&params, ptr, 2); ptr += 2;
                   bool bTrackingValid = params & 0x01; // 0x01 : rigid body was successfully tracked in this frame
   
                   rigid_body.track_valid = bTrackingValid;
                   ROS_DEBUG_STREAM("track valid  "<< (bTrackingValid ? "true": "false") );
               }
               natnet_data.rigid_bodies.push_back(rigid_body);
   
           } // Go to next rigid body
   
   
           // Skeletons (NatNet version 2.1 and later)
           if( ((major == 2)&&(minor>0)) || (major>2))
           {
               int nSkeletons = 0;
               memcpy(&nSkeletons, ptr, 4); ptr += 4;
   //            printf("Skeleton Count : %d\n", nSkeletons);
   
               ROS_DEBUG("Skeleton Count : %d ", nSkeletons);
   
               // Loop through skeletons
               for (int j=0; j < nSkeletons; j++)
               {
                   // skeleton id
                   int skeletonID = 0;
                   memcpy(&skeletonID, ptr, 4); ptr += 4;
   
                   // Number of rigid bodies (bones) in skeleton
                   int nRigidBodies = 0;
                   memcpy(&nRigidBodies, ptr, 4); ptr += 4;
   //                printf("Rigid Body Count : %d\n", nRigidBodies);
   
                   ROS_DEBUG("Skeleton ID %d Rigid Body Count : %d ", skeletonID, nRigidBodies);
   
                   natnet_bridge::Skeleton skeleton;
                   skeleton.id = skeletonID; skeleton.n_rigid_bodies = nRigidBodies;
   
                   // Loop through rigid bodies (bones) in skeleton
                   for (int j=0; j < nRigidBodies; j++)
                   {
                       // Rigid body position and orientation
                       int ID = 0; memcpy(&ID, ptr, 4); ptr += 4;
                       float x = 0.0f; memcpy(&x, ptr, 4); ptr += 4;
                       float y = 0.0f; memcpy(&y, ptr, 4); ptr += 4;
                       float z = 0.0f; memcpy(&z, ptr, 4); ptr += 4;
                       float qx = 0; memcpy(&qx, ptr, 4); ptr += 4;
                       float qy = 0; memcpy(&qy, ptr, 4); ptr += 4;
                       float qz = 0; memcpy(&qz, ptr, 4); ptr += 4;
                       float qw = 0; memcpy(&qw, ptr, 4); ptr += 4;
   //                    printf("ID : %d\n", ID);
   //                    printf("pos: [%3.2f,%3.2f,%3.2f]\n", x,y,z);
   //                    printf("ori: [%3.2f,%3.2f,%3.2f,%3.2f]\n", qx,qy,qz,qw);
                       ROS_DEBUG("ID : %d", ID);
                       ROS_DEBUG("pos: [%3.2f,%3.2f,%3.2f]", x,y,z);
                       ROS_DEBUG("ori: [%3.2f,%3.2f,%3.2f,%3.2f]", qx,qy,qz,qw);
   
                       natnet_bridge::RigidBody rigid_body_ske;
                       rigid_body_ske.id = ID;
                       rigid_body_ske.pose.position.x = x; rigid_body_ske.pose.position.y = y;
                       rigid_body_ske.pose.position.z = z;
                       rigid_body_ske.pose.orientation.x = qx; rigid_body_ske.pose.orientation.y = qy;
                       rigid_body_ske.pose.orientation.z = qz; rigid_body_ske.pose.orientation.w = qw;
   
                       // Mean marker error (NatNet version 2.0 and later)
                       if(major >= 2)
                       {
                           float fError = 0.0f; memcpy(&fError, ptr, 4); ptr += 4;
   //                        printf("Mean marker error: %3.2f\n", fError);
   
                           ROS_DEBUG("Mean marker error: %3.2f", fError);
                           rigid_body_ske.error = fError;
                       }
   
                       // Tracking flags (NatNet version 2.6 and later)
                       if( ((major == 2)&&(minor >= 6)) || (major > 2) || (major == 0) )
                       {
                           // params
                           short params = 0; memcpy(&params, ptr, 2); ptr += 2;
                           bool bTrackingValid = params & 0x01; // 0x01 : rigid body was successfully tracked in this frame
   
                           ROS_DEBUG_STREAM("track valid " << (bTrackingValid ? "true" : "false"));
                           rigid_body_ske.track_valid = bTrackingValid;
                       }
                       skeleton.rigid_bodies.push_back(rigid_body_ske);
   
                   } // next rigid body
                   natnet_data.skeletons.push_back(skeleton);
               } // next skeleton
           }
   
           // labeled markers (NatNet version 2.3 and later)
           // labeled markers - this includes all markers: Active, Passive, and 'unlabeled' (markers with no asset but a PointCloud ID)
           if( ((major == 2)&&(minor>=3)) || (major>2))
           {
               int nLabeledMarkers = 0;
               memcpy(&nLabeledMarkers, ptr, 4); ptr += 4;
   //            printf("Labeled Marker Count : %d\n", nLabeledMarkers);
               ROS_DEBUG("Labeled Marker Count : %d ", nLabeledMarkers);
   
               // Loop through labeled markers
               for (int j=0; j < nLabeledMarkers; j++)
               {
                   // id
                   // Marker ID Scheme:
                   // Active Markers:
                   //   ID = ActiveID, correlates to RB ActiveLabels list
                   // Passive Markers:
                   //   If Asset with Legacy Labels
                   //      AssetID     (Hi Word)
                   //      MemberID    (Lo Word)
                   //   Else
                   //      PointCloud ID
                   int ID = 0; memcpy(&ID, ptr, 4); ptr += 4;
                   int modelID, markerID;
                   DecodeMarkerID(ID, &modelID, &markerID);
   
   
                   // x
                   float x = 0.0f; memcpy(&x, ptr, 4); ptr += 4;
                   // y
                   float y = 0.0f; memcpy(&y, ptr, 4); ptr += 4;
                   // z
                   float z = 0.0f; memcpy(&z, ptr, 4); ptr += 4;
                   // size
                   float size = 0.0f; memcpy(&size, ptr, 4); ptr += 4;
   
                   natnet_bridge::LabeledMarker l_marker;
                   l_marker.b_unlabeled = false; // again ensure default is unlabeled marker
   
                   // NatNet version 2.6 and later
                   if( ((major == 2)&&(minor >= 6)) || (major > 2) || (major == 0) )
                   {
                       // marker params
                       short params = 0; memcpy(&params, ptr, 2); ptr += 2;
                       bool bOccluded = (params & 0x01) != 0;     // marker was not visible (occluded) in this frame
                       bool bPCSolved = (params & 0x02) != 0;     // position provided by point cloud solve
                       bool bModelSolved = (params & 0x04) != 0;  // position provided by model solve
                       l_marker.b_occluded = bOccluded; l_marker.b_pc_solved = bPCSolved;
                       l_marker.b_model_solved=bModelSolved;
                       if ((major >= 3) || (major == 0))
                       {
                           bool bHasModel = (params & 0x08) != 0;     // marker has an associated asset in the data stream
                           bool bUnlabeled = (params & 0x10) != 0;    // marker is 'unlabeled', but has a point cloud ID
                           bool bActiveMarker = (params & 0x20) != 0; // marker is an actively labeled LED marker
                           l_marker.b_has_model = bHasModel; l_marker.b_unlabeled = bUnlabeled;
                           l_marker.b_activeMarker = bActiveMarker;
                       }
   
                   }
   
                   // NatNet version 3.0 and later
                   float residual = 0.0f;
                   if ((major >= 3) || (major == 0))
                   {
                       // Marker residual
                       memcpy(&residual, ptr, 4); ptr += 4;
                   }
   
   //                printf("ID  : [MarkerID: %d] [ModelID: %d]\n", markerID, modelID);
   //                printf("pos : [%3.2f,%3.2f,%3.2f]\n", x,y,z);
   //                printf("size: [%3.2f]\n", size);
   //                printf("err:  [%3.2f]\n", residual);
                   ROS_DEBUG("ID  : [MarkerID: %d] [ModelID: %d]", markerID, modelID);
                   ROS_DEBUG("pos : [%3.2f,%3.2f,%3.2f]", x,y,z);
                   ROS_DEBUG("size: [%3.2f]", size);
                   ROS_DEBUG("err:  [%3.2f]", residual);
                   l_marker.marker_id = markerID; l_marker.model_id = modelID;
                   l_marker.position.x = x; l_marker.position.y = y; l_marker.position.z = z;
                   l_marker.size = size; l_marker.residual = residual;
   
                   if (!l_marker.b_unlabeled)
                       natnet_data.labeled_markers.push_back(l_marker);
                   else
                       natnet_data.unlabeled_markers.push_back(l_marker);
               }
           }
   
           // Not need to parse since it doesn't send for IAS
           // Force Plate data (NatNet version 2.9 and later)
           if (((major == 2) && (minor >= 9)) || (major > 2))
           {
               int nForcePlates;
               memcpy(&nForcePlates, ptr, 4); ptr += 4;
               for (int iForcePlate = 0; iForcePlate < nForcePlates; iForcePlate++)
               {
                   // ID
                   int ID = 0; memcpy(&ID, ptr, 4); ptr += 4;
                   printf("Force Plate : %d\n", ID);
   
                   // Channel Count
                   int nChannels = 0; memcpy(&nChannels, ptr, 4); ptr += 4;
   
                   // Channel Data
                   for (int i = 0; i < nChannels; i++)
                   {
                       printf(" Channel %d : ", i);
                       int nFrames = 0; memcpy(&nFrames, ptr, 4); ptr += 4;
                       for (int j = 0; j < nFrames; j++)
                       {
                           float val = 0.0f;  memcpy(&val, ptr, 4); ptr += 4;
                           printf("%3.2f   ", val);
                       }
                       printf("\n");
                   }
               }
           }
   
           // Not need to parse since it doesn't send for IAS
           // Device data (NatNet version 3.0 and later)
           if (((major == 2) && (minor >= 11)) || (major > 2))
           {
               int nDevices;
               memcpy(&nDevices, ptr, 4); ptr += 4;
               for (int iDevice = 0; iDevice < nDevices; iDevice++)
               {
                   // ID
                   int ID = 0; memcpy(&ID, ptr, 4); ptr += 4;
                   printf("Device : %d\n", ID);
   
                   // Channel Count
                   int nChannels = 0; memcpy(&nChannels, ptr, 4); ptr += 4;
   
                   // Channel Data
                   for (int i = 0; i < nChannels; i++)
                   {
                       printf(" Channel %d : ", i);
                       int nFrames = 0; memcpy(&nFrames, ptr, 4); ptr += 4;
                       for (int j = 0; j < nFrames; j++)
                       {
                           float val = 0.0f;  memcpy(&val, ptr, 4); ptr += 4;
                           printf("%3.2f   ", val);
                       }
                       printf("\n");
                   }
               }
           }
   
           // software latency (removed in version 3.0)
           if ( major < 3 )
           {
               float softwareLatency = 0.0f; memcpy(&softwareLatency, ptr, 4); ptr += 4;
   //            printf("software latency : %3.3f\n", softwareLatency);
               ROS_DEBUG("software latency : %3.3f\n", softwareLatency);
           }
   
           // timecode
           unsigned int timecode = 0;  memcpy(&timecode, ptr, 4);  ptr += 4;
           unsigned int timecodeSub = 0; memcpy(&timecodeSub, ptr, 4); ptr += 4;
           char szTimecode[128] = "";
           TimecodeStringify(timecode, timecodeSub, szTimecode, 128);
   
           // timestamp
           double timestamp = 0.0f;
   
           // NatNet version 2.7 and later - increased from single to double precision
           if( ((major == 2)&&(minor>=7)) || (major>2))
           {
               memcpy(&timestamp, ptr, 8); ptr += 8;
           }
           else
           {
               float fTemp = 0.0f;
               memcpy(&fTemp, ptr, 4); ptr += 4;
               timestamp = (double)fTemp;
           }
   //        printf("Timestamp : %3.3f\n", timestamp);
           ROS_DEBUG("Timestamp : %3.3f\n", timestamp);
   
           // high res timestamps (version 3.0 and later)
           if ( (major >= 3) || (major == 0) )
           {
               uint64_t cameraMidExposureTimestamp = 0;
               memcpy( &cameraMidExposureTimestamp, ptr, 8 ); ptr += 8;
   //            printf( "Mid-exposure timestamp : %" PRIu64"\n", cameraMidExposureTimestamp );
   
               uint64_t cameraDataReceivedTimestamp = 0;
               memcpy( &cameraDataReceivedTimestamp, ptr, 8 ); ptr += 8;
   //            printf( "Camera data received timestamp : %" PRIu64"\n", cameraDataReceivedTimestamp );
   
               uint64_t transmitTimestamp = 0;
               memcpy( &transmitTimestamp, ptr, 8 ); ptr += 8;
   //            printf( "Transmit timestamp : %" PRIu64"\n", transmitTimestamp );
   
               ROS_DEBUG( "Mid-exposure timestamp : %" PRIu64"\n", cameraMidExposureTimestamp );
               ROS_DEBUG( "Camera data received timestamp : %" PRIu64"\n", cameraDataReceivedTimestamp );
               ROS_DEBUG( "Transmit timestamp : %" PRIu64"\n", transmitTimestamp );
           }
   
           // frame params
           short params = 0;  memcpy(&params, ptr, 2); ptr += 2;
           bool bIsRecording = (params & 0x01) != 0;                  // 0x01 Motive is recording
           bool bTrackedModelsChanged = (params & 0x02) != 0;         // 0x02 Actively tracked model list has changed
   
   
           // end of data tag
           int eod = 0; memcpy(&eod, ptr, 4); ptr += 4;
           ROS_DEBUG("-----------------End Packet-------------");
   
       }
       else
       {
           ROS_WARN("Unrecognized Packet Type. Message ID: %d", MessageID);
       }
   
   }
   
   }
   
