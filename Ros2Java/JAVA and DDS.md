Usefully, there [exist JAVA bindings](https://github.com/OpenDDS/OpenDDS/tree/master/java)  for DDS (Specifically [OpenDDS](obsidian://open?vault=Obsidian&file=DDS%2FAttempting%20Open%20DDS%20on%20Windows)).

This provides a method for moving forward with the ROS 2 KUKA upgrade.

The benefit of building a bespoke DDS communication layer in JAVA for KUKA is that in theory the KUKA should remain relatively agnostic of the particular flavour of ROS2 used elsewhere.