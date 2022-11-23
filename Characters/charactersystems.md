
# Systems

While the main character update should be done by a user system, this package does come with a few pre-made systems that handle other important aspects of the character logic:
* `KinematicCharacterDeferredImpulsesSystem`: handles applying impulses stored in the `KinematicCharacterDeferredImpulse` buffer, after the character update.
* `StoreKinematicCharacterBodyPropertiesSystem`: handles storing character data in the `StoredKinematicCharacterData` component.
* `TrackedTransformFixedSimulationSystem`: handles storing "previous transform" data in all `TrackedTransform` components.
* `CharacterInterpolationRememberTransformSystem` and `CharacterInterpolationSystem`: handles character interpolation, if any.

There are also two pre-made system groups to be aware of:
* `KinematicCharacterPhysicsUpdateGroup`: provides a sensible default update point for fixed-rate character physics update systems (used by Standard Characters)
* `KinematicCharacterVariableUpdateGroup`: provides a sensible default update point for variable-rate character update systems (used by Standard Characters)