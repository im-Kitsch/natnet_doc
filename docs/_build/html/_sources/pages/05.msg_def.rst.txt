NatNet Message definition
=============================


To publish a complete natnet dataframe, we use a customized ros message named ``NatNetFrame`` . 
The structure of message followed the protocol of NatNet. You can check it via run::

    $ rosmsg show natnet_bridge/NatNetFrame


 


The documentation of ROS Style are here

--`Customized definition of natnet_pkg <./../_static/index-msg.html>`_
    -- `NatNetFrame <./../_static/index-msg.html>`_ 

    -- `MarkerSet <./../_static/index-msg.html>`_

    -- `LabeledMarker <./../_static/index-msg.html>`_

    -- `RigidBody <./../_static/index-msg.html>`_

    -- `Skeleton <./../_static/index-msg.html>`_


.. warning::
    Motive changes the interfaces time by time.
    So not all members in the ``NatNetFrame`` is supported for all natnet versions.
    Also not all information in protocol are parsed in NatNetFrame. E.g. the force plate is not 
    used so it is ignored. Also the software latency is not used, since Motive doesn't 
    have documentations about time definitions, so the time latency information is also given up. 
    
    So if you want to know more detail about the protocol, 
    you have to check function *unpack()* both in *natnet_bridge/src/unpack.cpp* and 
    in NatNet SDK(Windows)'s example, *Packet Client*, 
    which is not supported in Linux but there is only example on Windows Platform.

A concise description of message definition is   ::
   
    std_msgs/Header header #  standard for ros
        uint32 seq
        time stamp
        string frame_id
    string reference_frame  # default: 'optitrack', this is the 'world' frame of motive 
    int32 natnet_frame_number # defined by natnet
    natnet_bridge/MarkerSet[] marker_sets
        string name
        int32 n_markers
        natnet_bridge/PointFloat32[] markers
            float32 x, y, z
    natnet_bridge/PointFloat32[] unidentified_markers
        float32 x, y, z    
    natnet_bridge/RigidBody[] rigid_bodies
        int32 id
        natnet_bridge/PoseFloat32 pose
            natnet_bridge/PointFloat32 position
                float32 x, y, z           
            natnet_bridge/QuaternionFloat32 orientation
                float32 x, y, z, w
        float32 error
        bool track_valid
    natnet_bridge/Skeleton[] skeletons
        int32 id
        int32 n_rigid_bodies
        natnet_bridge/RigidBody[] rigid_bodies
            int32 id
            natnet_bridge/PoseFloat32 pose
                natnet_bridge/PointFloat32 position
                    float32 x, y, z     
                natnet_bridge/QuaternionFloat32 orientation
                    float32 x, y, z, w                
            float32 error
            bool track_valid
    natnet_bridge/LabeledMarker[] labeled_markers
    natnet_bridge/LabeledMarker[] unlabeled_markers
        int32 model_id
        int32 marker_id
        natnet_bridge/PointFloat32 position
            float32 x, y, z
        float32 size
        bool b_occluded
        bool b_pc_solved
        bool b_model_solved
        bool b_has_model
        bool b_unlabeled
        bool b_activeMarker
        float32 residual

