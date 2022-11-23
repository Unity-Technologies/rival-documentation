
## v0.3 Upgrade Tips - DetectFutureSlopeChange

As part of a fix for the implementation of `PreventGroundingWhenMovingTowardsNoGrounding` in standard characters, the call to `KinematicCharacterUtilities.DefaultMethods.DetectFutureSlopeChange()` has been **removed** from `characterProcessor.IsGroundedOnHit()`, and **moved** to `characterProcessor.OnUpdate()`. If you already had an existing character before upgrading, you may want to apply this fix to your character as well.

The code samples below are using the `FirstPersonCharacter` as an example, but you can simply replace `FirstPersonCharacter` with whatever name you chose for your character component.

Your `IsGroundedOnHit` should now look like this (the whole block of code in comments is what **must be removed**):
```cs
    public bool IsGroundedOnHit(in BasicHit hit, int groundingEvaluationType)
    {
        // Prevent grounding from slope change
        // if (FirstPersonCharacter.PreventGroundingWhenMovingTowardsNoGrounding || FirstPersonCharacter.HasMaxDownwardSlopeChangeAngle)
        // {
        //     KinematicCharacterUtilities.DefaultMethods.DetectFutureSlopeChange(
        //         ref this,
        //         in hit,
        //         in CharacterBody,
        //         in PhysicsCollider,
        //         Entity,
        //         CharacterBody.RelativeVelocity,
        //         GroundingUp,
        //         0.05f, // verticalOffset
        //         0.05f, // downDetectionDepth
        //         DeltaTime, // deltaTimeIntoFuture
        //         0.25f, // secondaryNoGroundingCheckDistance
        //         FirstPersonCharacter.StepHandling,
        //         FirstPersonCharacter.MaxStepHeight,
        //         out bool isMovingTowardsNoGrounding,
        //         out bool foundSlopeHit,
        //         out float futureSlopeChangeAnglesRadians,
        //         out RaycastHit futureSlopeHit);
        //     if (FirstPersonCharacter.PreventGroundingWhenMovingTowardsNoGrounding && isMovingTowardsNoGrounding)
        //     {
        //         return false;
        //     }
        //     if (FirstPersonCharacter.HasMaxDownwardSlopeChangeAngle && foundSlopeHit && math.degrees(futureSlopeChangeAnglesRadians) < // -FirstPersonCharacter.MaxDownwardSlopeChangeAngle)
        //     {
        //         return false;
        //     }
        // }

        return KinematicCharacterUtilities.DefaultMethods.IsGroundedOnHit(
            ref this,
            in hit,
            in CharacterBody,
            in PhysicsCollider,
            Entity,
            GroundingUp,
            groundingEvaluationType,
            FirstPersonCharacter.StepHandling,
            FirstPersonCharacter.MaxStepHeight,
            FirstPersonCharacter.ExtraStepChecksDistance);
    }
```

and your `OnUpdate()` should now look like this (the block of code under the `// BEGIN` and `// END` comments is what **must be added**. It's important that this code be placed after `HandleCharacterControl()`, and before `MovementAndDecollisionsUpdate()`):
```cs
    public unsafe void OnUpdate()
    {
        GroundingUp = -math.normalizesafe(FirstPersonCharacter.Gravity);

        KinematicCharacterUtilities.InitializationUpdate(ref CharacterBody, ref CharacterHitsBuffer, ref VelocityProjectionHitsBuffer, ref CharacterDeferredImpulsesBuffer);
        KinematicCharacterUtilities.ParentMovementUpdate(ref this, ref Translation, ref CharacterBody, in PhysicsCollider, DeltaTime, Entity, Rotation, GroundingUp, CharacterBody.WasGroundedBeforeCharacterUpdate); // safe to remove if not needed
        KinematicCharacterUtilities.GroundingUpdate(ref this, ref Translation, ref CharacterBody, ref CharacterHitsBuffer, ref VelocityProjectionHitsBuffer, in PhysicsCollider, Entity, Rotation, GroundingUp);

        // Character velocity control is updated AFTER the ground has been detected, but BEFORE the character tries to move & collide with that velocity
        HandleCharacterControl();

        // BEGIN
        // Prevent grounding from future slope change
        if (CharacterBody.IsGrounded && (FirstPersonCharacter.PreventGroundingWhenMovingTowardsNoGrounding || FirstPersonCharacter.HasMaxDownwardSlopeChangeAngle))
        {
            KinematicCharacterUtilities.DefaultMethods.DetectFutureSlopeChange(
                ref this,
                in CharacterBody.GroundHit,
                in CharacterBody,
                in PhysicsCollider,
                Entity,
                CharacterBody.RelativeVelocity,
                GroundingUp,
                0.05f, // verticalOffset
                0.05f, // downDetectionDepth
                DeltaTime, // deltaTimeIntoFuture
                0.25f, // secondaryNoGroundingCheckDistance
                FirstPersonCharacter.StepHandling,
                FirstPersonCharacter.MaxStepHeight,
                out bool isMovingTowardsNoGrounding,
                out bool foundSlopeHit,
                out float futureSlopeChangeAnglesRadians,
                out RaycastHit futureSlopeHit);
            if ((FirstPersonCharacter.PreventGroundingWhenMovingTowardsNoGrounding && isMovingTowardsNoGrounding) ||
                (FirstPersonCharacter.HasMaxDownwardSlopeChangeAngle && foundSlopeHit && math.degrees(futureSlopeChangeAnglesRadians) < -FirstPersonCharacter.MaxDownwardSlopeChangeAngle))
            {
                CharacterBody.IsGrounded = false;
            }
        }
        // END

        if(CharacterBody.IsGrounded && CharacterBody.SimulateDynamicBody)
        {
            KinematicCharacterUtilities.DefaultMethods.UpdateGroundPushing(ref CollisionWorld, ref PhysicsMassFromEntity, ref CharacterDeferredImpulsesBuffer, ref CharacterBody, DeltaTime, FirstPersonCharacter.Gravity, 1f); // safe to remove if not needed
        }

        KinematicCharacterUtilities.MovementAndDecollisionsUpdate(ref this, ref Translation, ref CharacterBody, ref CharacterHitsBuffer, ref VelocityProjectionHitsBuffer, ref CharacterDeferredImpulsesBuffer, in PhysicsCollider, DeltaTime, Entity, Rotation, GroundingUp);
        KinematicCharacterUtilities.DefaultMethods.MovingPlatformDetection(ref TrackedTransformFromEntity, ref CharacterBodyFromEntity, ref CharacterBody); // safe to remove if not needed
        KinematicCharacterUtilities.ParentMomentumUpdate(ref TrackedTransformFromEntity, ref CharacterBody, in Translation, DeltaTime, GroundingUp); // safe to remove if not needed
        KinematicCharacterUtilities.ProcessStatefulCharacterHits(ref StatefulCharacterHitsBuffer, in CharacterHitsBuffer); // safe to remove if not needed
    }
```