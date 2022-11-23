
# Dynamic Body Interactions
Allowing your character to push and be pushed by dynamic rigidbodies can be achieved by enabling the `SimulateDynamicBody` option in your character's authoring component (or `KinematicCharacterProperties.SimulateDynamicBody` if you want to change it at runtime). When this is enabled, the character will apply force on itself and other bodies in order to immitate the behaviour of a true dynamic rigidbody. This uses the `Mass` property of the character's authoring component (or `KinematicCharacterProperties.Mass`) to simulate collision mass ratios.

Since the character is still a kinematic rigidbody with collisions, a dynamic rigidbody trying to push the character will still be stopped by the character as soon as the collision happens. In order to solve this, you need to make sure that dynamic bodies will not be stopped by collisions with the character. One way to achieve this is to set the character's PhysicsShape's collision response to either `None` or `RaiseTriggerEvents`. But you must remember this when doing any raycasts in your game that expect to hit characters.


## SynchronizeCollisionWorld

When dealing with a character that can push or be pushed by other bodies (kinematic or dynamic), you may want to add a `PhysicsStep` component to an entity in your scene, and set `SynchronizeCollisionWorld` to true. This will make sure that the `CollisionWorld` that the character update uses for physics queries is properly updated after the physics systems make the rigidbodies move. The result is that enabling `SynchronizeCollisionWorld` will get rid of some slight visible "lag" between the character and the object it pushes.