
# IKinematicCharacterProcessor

`IKinematicCharacterProcessor` is an interface that allows users to customize some of the logic that happens during the internal character updates. It asks users to define the implementation of several methods that will be called during the character update. Most of these methods should call a "default" implementation that can be found in the `KinematicCharacterAspect`, but asking users to call these manually opens the door for more customizability. You can think of this approach as an alternative to virtual methods with child overrides (which we cannot have in bursted jobs).

Here is a description of these methods:
- `UpdateGroundingUp`: asks to update the direction that our character considers as its "up" direction to compare slope normals with. This direction must be written to `KinematicCharacterBody.GroundingUp`.
- `CanCollideWithHit`: asks to return true if the character should be able to collide with the hit, and return false if not.
- `OnMovementHit`: asks to determine what happens when the character collider casts have detected a hit during the movement iterations. By default, this should call `KinematicCharacterAspect.Default_OnMovementHit`.
- `IsGroundedOnHit`: asks to determine if the character is grounded on the hit or not. By default, we call `KinematicCharacterAspect.Default_IsGroundedOnHit`, which will check the slope angle and velocity direction in order to determine the final result.
- `OverrideDynamicHitMasses`: gives users an opportunity to modify the mass ratios between the character and another dynamic body when they are colliding. This is only called for characters that have `KinematicCharacterProperties.SimulateDynamicBody` set to true. This function can be left empty if you do not wish to modify anything.
- `ProjectVelocityOnHits`: asks to determine how the character velocity gets projected on hits, based on all hits so far this frame. By default, we should call `KinematicCharacterAspect.Default_ProjectVelocityOnHits` here. This is callback mostly for advanced usage, and it is recommended not to change it unless you wish to make your character bounce on certain surfaces, for example.

It is recommended to carefully study & understand how the "Standard Characters" implement these before attempting to modify them. **Changing some of these things could break the character movement** or have a negative impact on performance.
