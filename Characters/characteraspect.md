
# Character Aspect

The `KinematicCharacterAspect` is where the large majority of the core character update happens. It's an aspect that includes all of the core character components, and all of the core methods to move the character & query its environment. A non-exhaustive list of its methods follows.

## Main character update steps
These are the main steps for a typical character update. They are called in the correct order by the character aspect's `PhysicsUpdate` in the "Standard Characters":
* `Update_Initialize`: clears & initializes some core character data and buffers at the start of the update.
* `Update_ParentMovement`: handles moving the character based on its currently-assigned `KinematicCharacterBody.ParentEntity`, if any.
* `Update_Grounding`: handles detecting character grounding.
* `Update_PreventGroundingFromFutureSlopeChange`: handles canceling grounded status depending on what has been defined in the `BasicStepAndSlopeHandlingParameters` given to the method (for example; cancel grounding if we're headed towards a ledge).
* `Update_GroundPushing`: handles applying a constant force to our current ground entity, if that entity is dynamic.
* `Update_MovementAndDecollisions`: handles moving the character with its velocity and solving collisions.
* `Update_MovingPlatformDetection`: handles detecting valid moving platform entities, and assigning them as our character's `KinematicCharacterBody.ParentEntity`.
* `Update_ParentMomentum`: handles preserving velocity momentum when getting unparented from a parent body.
* `Update_ProcessStatefulCharacterHits`: handles filling the `StatefulKinematicCharacterHit` buffer on the character entity, with character hits that have an Enter/Exit/Stay state.


## Queries
The character aspect includes several methods for doing physics queries that understand how to filter out hits that the character shouldn't be able to detect, based on the implementation of `IKinematicCharacterProcessor.CanCollideWithHit`:
* `CastColliderClosestCollisions`
* `CastColliderAllCollisions`
* `RaycastClosestCollisions`
* `RaycastAllCollisions`
* `CalculateDistanceClosestCollisions`
* `CalculateDistanceAllCollisions`


## Default IKinematicCharacterProcessor method implementations
The character aspect includes default implementations for several of the `IKinematicCharacterProcessor` methods:
* `Default_IsGroundedOnHit`: determines grounding on hit based on slope, velocity, step handling parameters, etc...
* `Default_UpdateGroundingUp`: sets grounding up to character transform up.
* `Default_ProjectVelocityOnHits`: does a velocity projection based on the current ground status of the character, and the ground status of the detected hit(s).
* `Default_OnMovementHit`: handles moving to hit, projecting velocity, stepping up, etc...


## Other
* `SetOrUpdateParentBody`: handles the computations required to properly update the character's `ParentEntity` (along with information about the "anchor point" of the character on the parent).
* `DetectFutureSlopeChange`: calculated information about the incoming slope changes based on the current character velocity.
* `IsGroundedOnSlopeNormal`: determines if we'd be grounded on a hit normal based on slope angle alone.
* `CalculateAngleOfHitWithGroundUp`: determines the "slope angle" of a hit based on its normal and the character grounding up direction.
* `MovementWouldHitNonGroundedObstruction`: determines if a certain character movement would end up detecting any ungrounded hits. Can be useful to prevent air-climbing some slopes when the character has a high air acceleration.