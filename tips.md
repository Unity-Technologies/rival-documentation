
# Performance Tips

- **Convex level colliders and low collider polygon counts**: For the level geometry, use convex collider shapes whenever possible, and make sure your non-convex colliders have as little polygons as possible. It is standard practice in games to create simplified collision meshes for mesh objects.
- **Disabling the physics step**: This character doesn't need any physics step in order to function properly. You can safely set the "Simulation Type" to "None" on the Physics Step component, if you don't need any physics step for physics bodies other than characters.
- **Small ground probing/snap distances**: In order to detect grounding, the character does a downward collider cast with the length `KinematicCharacterProperties.GroundSnappingDistance`. You should keep this value as small as possible to get the job done, because the longer the collider cast, the heavier it can be to process. 
- **Character iteration counts**: Don't assign iteration counts that are too high for `KinematicCharacterProperties.MaxContinuousCollisionsIterations` and `KinematicCharacterProperties.MaxOverlapDecollisionIterations`. If in doubt, keep the defaults
- **`KinematicCharacterProperties.ProjectVelocityOnInitialOverlaps`**: Enabling this option on the character body adds one CalculateDistance call per character. Only activate it if you need it (it can help prevent tunnelling when your character has a shape that changes its collisions when it is rotated).
- **`KinematicCharacterProperties.DetectObstructionsForParentBodyMovement`**: Enabling this option on the character body adds one CastCollider call per character when the character has a "ParentEntity" set.
- **Slope Angle Changes Detection**: Enabling either `PreventGroundingWhenMovingTowardsNoGrounding` or `HasMaxDownwardSlopeChangeAngle` on the standard characters has a performance impact, because it adds a few raycast queries per character. Make sure you only use this when necessary.
- **Step Handling**: Enabling step handling on the standard characters adds the cost of potentially up to 3 collider casts on each hit that is considered non-grounded, and a few additional raycasts. Keep this in mind.
