
# Networking

(for a full example of predicted character networking, see the OnlineFPS sample)

Every game's needs are different, and therefore, the steps required to network a character controller will vary from one game to another. However, we could argue that one of the most widespread approaches to networking a character controller will involve:
* Client-side prediction on the client that owns the character
* Interpolation on clients that don't own the character

This article will go over the necessary steps for implementing this approach, using the `NetCode` package.


## Ghost component setup
* Your Player and Character prefabs both need a `GhostAuthoringComponent`, with `Default Ghost Mode` set to `Owner Predicted` and `Has Owner` set to `true`
* Your Player prefab's `GhostAuthoringComponent` also needs `Support Auto Command Target` and `Track Interpolation Delay` set to true


## What to synchronize on the character
The following fields on the character entity need to be synchronized over network, using `[GhostField]`:
* `LocalTransform.Position`
* `LocalTransform.Rotation` (depending on the game, you may want to synchronize only the Y euler angle instead of the full rotation, and you'd have to reconstruct the full rotation from that angle in a prediction system)
* `KinematicCharacterBody.IsGrounded`
* `KinematicCharacterBody.RelativeVelocity`
* (only if using the First-Person character) `FirstPersonCharacterComponent.ViewPitchDegrees`
* (only if you use the ParentEntity mechanism in your game) `KinematicCharacterBody.ParentEntity` and `KinematicCharacterBody.ParentLocalAnchorPoint`

This constitutes all of the basic required character state data to synchronize over network. However, you must remember that any character feature you add may or may not require additional data to synchronize.

It's also worth noting that the `PhysicsVelocity` component on the character entity must **not** be synchronized.


## Prediction
Certain changes are necessary in order to enable prediction for all standard character & player systems:
* `First/ThirdPersonCharacterVariableUpdateSystem` need `[UpdateInGroup(typeof(PredictedSimulationSystemGroup))]` and `[UpdateAfter(typeof(PredictedFixedStepSimulationSystemGroup))]`
* `First/ThirdPersonPlayerInputsSystem` need `[UpdateInGroup(typeof(GhostInputSystemGroup))]`
* `First/ThirdPersonPlayerVariableStepControlSystem` need `[UpdateInGroup(typeof(PredictedSimulationSystemGroup))]` and `[UpdateBefore(typeof(First/ThirdPersonCharacterVariableUpdateSystem))]`
* `First/ThirdPersonPlayerFixedStepControlSystem` need `[UpdateInGroup(typeof(PredictedFixedStepSimulationSystemGroup), OrderFirst = true)]`

`First/ThirdPersonCharacterPhysicsUpdateSystem` don't need to change, because they are part of the physics update group, which is automatically moved to the prediction loop by NetCode.


## Player commands
You must replace input handling in all of your player systems with NetCode commands. It is recommended that you use the `IInputComponentData` approach, which provides easy support for button press events using `InputEvent`.

* `First/ThirdPersonPlayerInputsSystem` will become responsible for storing player commands in your `IInputComponentData` component or in your commands buffer.
* `First/ThirdPersonPlayerVariableStepControlSystem` and `First/ThirdPersonPlayerFixedStepControlSystem` will become responsible for getting commands from the appropriate prediction tick, and applying them to the controlled character.


## Character interpolation
There are a few particularities to keep in mind when it comes to character interpolation in a netcode context:
* On clients, client-owned characters will be predicted and will update at a fixed timestep in the fixed step prediction group; this means they need character interpolation.
* On clients, remote characters will be interpolated by NetCode; this means they must not have the built-in character interpolation.
* On server, there must be no interpolation because there is no presentation.

Because of this situation, we need the `CharacterInterpolation` component to only be present on clients, and on characters that are predicted. The following code can be copied into your project, and will take care of making character interpolation compatible with this networking approach:

```cs
[GhostComponentVariation(typeof(CharacterInterpolation))]
[GhostComponent(PrefabType = GhostPrefabType.PredictedClient)]
public struct CharacterInterpolation_GhostVariant
{ }
```