
# Utilities

## KinematicCharacterUtilities
`KinematicCharacterUtilities` contains various helper methods related to characters:
* `GetBaseCharacterQueryBuilder`: returns an `EntityQueryBuilder` that includes all core character components (and can be further added-to by users).
* `CreateCharacter`: handles adding all of the required character components to an entity at runtime.
* `BakeCharacter`: handles adding all of the required character components to an entity during baking.
* `AddVariableRateRotationFromFixedRateRotation`: handles updating a rotation at a variable rate, based on a reference rotation that has calculated for a fixed rate. This is mostly used to apply a character's parent's rotation to the character itself or the camera
* `IsHitCollidableOrCharacter`: characters may need to be triggers instead of colliders if they require interaction with dynamic rigidbodies, and so this method allows for an easy way to check if a physics query has hit either a collidable collider or a character.


## PhysicsUtilities
`PhysicsUtilities` contains various physics helper functions.


## MathUtilities
`MathUtilities` contains various math helper functions.


## CharacterControlUtilities
`CharacterControlUtilities` contains various methods related to controlling the character velocity & rotation.