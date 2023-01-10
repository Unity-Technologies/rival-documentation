
# Changelog

[Previous Versions](./Changelog/changelog-archive.md)

## v1.0.0-pre.15.1

**Upgrade Tips:**
* A `CharacterWidthForStepGroundingCheck` field was added to the `BasicStepAndSlopeHandlingParameters` struct. This field is configurable in the standard character authoring components. If you are upgrading from a previous version, you should take care of passing that value as the final (optional) parameter to `CharacterAspect.Default_OnMovementHit` in your character aspect. Look for the `// HERE` comment in the snippet below:
```cs
    public void OnMovementHit(
            ref ThirdPersonCharacterUpdateContext context,
            ref KinematicCharacterUpdateContext baseContext,
            ref KinematicCharacterHit hit,
            ref float3 remainingMovementDirection,
            ref float remainingMovementLength,
            float3 originalVelocityDirection,
            float hitDistance)
    {
        ref KinematicCharacterBody characterBody = ref CharacterAspect.CharacterBody.ValueRW;
        ref float3 characterPosition = ref CharacterAspect.LocalTransform.ValueRW.Position;
        ThirdPersonCharacterComponent characterComponent = CharacterComponent.ValueRO;
        
        CharacterAspect.Default_OnMovementHit(
            in this,
            ref context,
            ref baseContext,
            ref characterBody,
            ref characterPosition,
            ref hit,
            ref remainingMovementDirection,
            ref remainingMovementLength,
            originalVelocityDirection,
            hitDistance,
            characterComponent.StepAndSlopeHandling.StepHandling,
            characterComponent.StepAndSlopeHandling.MaxStepHeight,
            characterComponent.StepAndSlopeHandling.CharacterWidthForStepGroundingCheck); // HERE
    }
```


**Changes:**
* Made certain standard character systems require the `FixedTickSystem.Singleton` to exist in order to run their updates, preventing a potential error if the singleton doesn't exist yet.
* Fixed an issue where character standing on dynamic rigidbodies without a `TrackedTransform` component would apply an incorrect force to the rigidbody (making it look like the character was attracting the rigidbody towards itself).
* Fixed several issues related to step handling:
    * Sometimes the character would fail to step onto a surface
    * Sometimes the character velocity would get deviated from its course when stepping
    * Dynamic bodies moving onto the character could create buggy behaviour when the character was stepping onto them as a result, and so step handling is now disabled on dynamic bodies
    * Sometimes the character would launch into the air when stepping onto an angled slope. A `CharacterWidthForStepGroundingCheck` field was added to the `BasicStepAndSlopeHandlingParameters` struct in order to solve this. This field is meant to represent the width of the character shape, and so for a capsule character, this width should be capsule radius x2. The system cannot infer this value automatically, because the character can support any convex physics shape in any orientation


**Known issues:**
* The `WorldTransform` component of the character does not get updated when character interpolation is enabled. Regular rigidbodies have the same issue whenever graphical smoothing is enabled, and so a more general fix is being investigated for this issue. In the meantime, you can use `LocalTransform` instead since character entitie are always root entities. An alternative would be to add a line to set the `WorldTransform` values to the `LocalTransform` values at the end of your character update in your character aspect.
* A known issue with transforms v2 prevents proper baking of entities that have both a non-unit scale and physics graphical smoothing (interpolation). In this situation, baking attempts to add the same component twice, resulting in a baking error. As a result, in sample projects, interpolation is disabled and jitter may be visible when a character stands on a moving physics body or when pushing against one.