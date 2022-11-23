
# Standard Characters Implementation Overview

We have 2 main prefabs: 
- Player (`ThirdPersonPlayer`, `FirstPersonPlayer`): Represents the human gathering input and controlling the Character (and Camera, if applicable)
- Character (`ThirdPersonCharacter`, `FirstPersonCharacter`): Represents the character that moves around in the world


## How does character control work?

The general idea here is:
- the Player gathers raw input from Unity's input systems
- the Player converts that raw input to a format that the Character can easily work with, and writes that data to the Control component (`ThirdPersonCharacterControl` or `FirstPersonCharacterControl`) on the character. This "Control component" is essentially meant to act as a common interface for controlling the character, whether it's a human or an AI controlling it. Both would be writing to this component.
- the Character moves with the input stored in its Control component

Let's take a closer look at how this happens in code, using the Third-Person Character as an example (the First-Person Character mostly works the same):

**Player**
1. `ThirdPersonPlayerInputsSystem` gathers input from Unity's input system every frame. It iterates on all entities with a `ThirdPersonPlayerInputs` component, and writes the raw input to that component. 
1. `ThirdPersonPlayerVariableStepControlSystem` and `ThirdPersonPlayerFixedStepControlSystem` handle taking the raw input from `ThirdPersonPlayerInputs`, and converting it to a format that the character can work with in the `ThirdPersonCharacterControl` component. They do this at variable and fixed update respectively, depending on when these inputs are actually used by the character.
1. The same is done for camera control; we write into the `OrbitCameraControl` component in order to tell the camera what to do.

**Character Movement (velocity)**
1. `ThirdPersonCharacterPhysicsUpdateSystem` updates at a fixed timestep, and schedules a job that calls `ThirdPersonCharacterAspect.PhysicsUpdate`.
1. `ThirdPersonCharacterAspect.PhysicsUpdate` calls the first phase of character update steps through the `CharacterAspect`, and then calls `HandleVelocityControl`.
1. `HandleVelocityControl` is where the `ThirdPersonCharacterControl` component is used to determine character velocity. This is the main function you should be modifying in order to customize character movement.
1. `ThirdPersonCharacterAspect.PhysicsUpdate` then calls the second phase of character update steps through the `CharacterAspect`, and it's in these steps that the character will actually move & detect collisions based on its velocity

**Character Rotation**
1. `ThirdPersonCharacterVariableUpdateSystem` updates at a variable timestep, and schedules a job that calls `ThirdPersonCharacterAspect.VariableUpdate`.
1. `ThirdPersonCharacterAspect.VariableUpdate` use the move vector of the `ThirdPersonCharacterControl` component in order to calculate and set a character rotation that faces this move direction.

**Camera**
1. `OrbitCameraSystem` moves the camera based on the look input in the `OrbitCameraControl` component.


## How to customize character movement?

### Velocity

As mentioned in the previous section, character velocity control is mainly implemented in your character aspect's `HandleVelocityControl` function. Here's a summary of what it does:
```cs
void HandleVelocityControl()
{
    // if the character is grounded
        // Move on ground by interpolating velocity towards a target max velocity
        // Handle jumping if there is jump input
    // otherwise, if the character is in air
        // Move in air by accelerating velocity towards a target max velocity, but also prevent that acceleration if we're moving against a steep wall
        // Apply gravity to character's velocity
        // Apply drag to character's velocity
}
```
This implementation is nothing more than a default starting point. Feel free to customize, change or replace anything in `HandleVelocityControl` in order to implement any character movement feature you need for your game.

### Rotation

As for character rotation control, it is mainly implemented in your character aspect's `VariableUpdate`. Once again, this is a default starting point that you could fully customize to suit your needs.


## Why are velocity & rotation control handled in different places?

In short; it's because velocity and rotation have a different ideal time when they should be updated.

Velocity is best handled at a fixed timestep, because it will directly participate in collisions & physics. The fixed timestep is ideal for such things because of its consistency and predictability.

Rotation is often best handled at a variable timestep (in sync with the rendering rate), because if it was handled at a fixed timestep with interpolation instead, there would be a noticeable lag/unresponsiveness with the "look rotation input". This is especially true for first-person characters where look rotation is controlled with the mouse, or with third-person characters that can aim directly in the camera direction while it rotates. Rotation can of course also participate in collisions & physics in certain cases (like when you rotate your character capsule on an axis other than Y), but this will typically have much less of an impact on physics solving than the velocity does. In any case, you should know that the decision to handle rotation at a variable rate is not a hard requirement; but only a default suggestion. If you feel like rotation would be better handled at a fixed timestep for your game, then you can move the rotation-handling code to your character aspect's `PhysicsUpdate` instead, and enable rotation interpolation in your character authoring. 


## Why separate Player and Character?

There are a few reasons why "character" and "player" are two different entities, two different sets of components, and two different sets of systems in the standard characters:
1. The same Character prefab/system should be controllable by either a human player or an AI controller. For this reason, we want the character prefab to be independent of any notion of who is controlling it. Sometimes the controller will be a human player (like when we assign the Character prefab as the "Controlled Character" of the Player prefab), but other times the same character prefab could be assigned to an AI controller. This is why the character "Control" component exists (`First/ThirdPersonCharacterControl`). It essentially acts as a common interface for controlling the character, whether the controller is human or AI.
1. It wouldn't make sense for an AI character to have any knowledge of a "Camera", but a human-controlled character must often move relatively to the camera orientation. For this reason, calculating a "camera-relative move input" is the repsonsibility of the system that handles the human Player logic; not of the Character systems. The human Player knows about the camera, and feeds camera-relative input to its controlled character. That way, the character can move relatively to the camera, without ever having to keep any kind of reference to a camera.
1. There are often cases where we need to destroy our Character entity, but we want the concept of our "Player" to survive. For example, in an online shooter game, you may have your character die, but your Player (score, name, money, unlockables, etc...) persists while awaiting the respawning of your Character. In these cases, because Player and Character are separate entities, we can easily just destroy the character entity, spawn a new one, and assign that new character as the "controlled character" of the Player.
1. At runtime, you might want to be able to switch the entity that your player controls. You could switch characters, or you could switch to controlling a horse or vehicle. When you switch the controlled entity like this, you probably still want to preserve data belonging to the player controlling the entity. The separation of Player and Character once again makes cases like these easier to deal with.

However, this is by no means a hard requirement for things to work. If you want to, you can change things so that the character entity & systems also handles player input directly.


## Why is Player input implemented this way?

You may notice a few peculiar things about the way player input is implemented:
* It is split into 3 different systems.
* It uses  a `FixedInputEvent` struct for button press input events, instead of simply using a boolean.
* The player writes input to their controlled character, instead of the character systems querying inputs directly.

While it would be possible to make things work with a simpler structure, things are structured this way due to the combination of several different considerations:
* Some inputs need processing at a variable rate, and some need processing at a fixed rate.
* There are certain challenges related to handling input that must be processed at a fixed timestep. You can read the [Input Handling](../../How_To/input-handling.md) article in order to better understand these challenges.
* As explained in the previous section, this structure allows characters to be controlled by either humans or AI, without requiring different implementations based on who or what controls them.
* This structure allows characters to be already setup for easy netcode compatibility (storing player commmands and using player commands are handled in different systems).

