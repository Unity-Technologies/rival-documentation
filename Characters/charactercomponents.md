
# Character Components

## KinematicCharacterProperties
`KinematicCharacterProperties` contains the character data that defines how the character behaves. Nothing in the character update will write to this component; it only reads from it. 

Examples of fields contained in this component:
* `InterpolateTranslation` and `InterpolateRotation`: should the character movement be interpolated?
* `EvaluateGrounding`: should the character detect & evaluate grounding?
* `MaxContinuousCollisionsIterations` how many collider cast iterations should the character movement update do before it breaks out of the loop?
* `SimulateDynamicBody`: should the character simulate forces when interacting with dynamic rigidbodies
* etc...


## KinematicCharacterBody
`KinematicCharacterBody` contains the character data that gets calculated & written to by the character update, and by the user.

Examples of fields contained in this component:
* `IsGrounded` and `GroundHit`: information about the grounding that was detected during the update
* `RelativeVelocity`: what is the current velocity of our character, relatively to its parent (if any)?
* `ParentEntity`: what is the Entity that the character considers as its current parent (this is typically used for moving platforms)?
* etc...


## StoredKinematicCharacterData
`StoredKinematicCharacterData` is a component that users will rarely have to interact with, but it could still be useful to understand why it exists. Essentially; it's a component that enables the character update to be done safely in parallel and in a deterministic way. The character update will sometimes need to access some data on other character entities (such as `Mass` or `RelativeVelocity`), and this data may change during each character's update. Therefore, in order to allow determinism, it is important to "store" a snapshot of all that character data before we run the character updates. `StoredKinematicCharacterData` is where we store that data. Without this, we could be accessing the `RelativeVelocity` of another character for example, but depending on which thread handles the update of that other character, that `RelativeVelocity` might get written to either before or after we attempt to access it; resulting in non-determinism.

This is for information purposes only. Users don't have to manually setup anything for this to happen, and the `StoredKinematicCharacterData` does its job automatically.


## Character DynamicBuffers
The character entity includes several DynamicBuffers in order to store information during its update:
* `KinematicCharacterHit`: all hits that were detected during the character update
* `StatefulKinematicCharacterHit`: all hits that were detected during the character update, but with state information (Enter/Exit/Stay)
* `KinematicCharacterDeferredImpulse`: stores impulses during the character update, and is used to apply them later on a single thread
* `KinematicVelocityProjectionHit`: stores all hits that should participate in the character's velocity projection on obstacles


## Other
* `TrackedTransform`: This component should be added to any moving entity that can be assigned as the `ParentEntity` of the character