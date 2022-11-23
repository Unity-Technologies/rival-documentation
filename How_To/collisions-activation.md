
# Character Collisions Activation

Completely deactivating the collision handling of a character requires more than simply setting the character's PhysicsCollider's collision response to "None". Setting the collision response to "None" prevents other rigidbodies from colliding with the character, but it does not prevent the character from detecting its own collisions with physics queries.

In order to completely deactivate character collisions, there are a few extra options to disable:
* set `KinematicCharacterProperties.EvaluateGrounding` to `false`
* set `KinematicCharacterProperties.DetectMovementCollisions` to `false`
* set `KinematicCharacterProperties.DecollideFromOverlaps` to `false`