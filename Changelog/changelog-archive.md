


## v1.0.0-pre.15

**Upgrade Tips:**
* The changes in this version should be considered too significant for a direct upgrade to be possible. Instead, it is recommended that you manually transfer the contents of your old character "Processor" to the new character Aspect


**Changes:**
* Major refactoring of the character code & structure in order to take advantage of new 1.0 features:
    * Usage of bursted `ISystem`
    * Character implementation is done with Aspects
    * Access to `PhysicsWorld` is done through singleton
    * Use transforms v2
    * etc...
* Major refactoring of the Rival Samples in order to reflect the changes in the core packages.


**Known issues:**
* A known issue with transforms v2 prevents proper baking of entities that have both a non-unit scale and physics graphical smoothing (interpolation). In this situation, baking attempts to add the same component twice, resulting in a baking error. As a result, in sample projects, interpolation is disabled and jitter may be visible when a character stands on a moving physics body or when pushing against one.


## v0.5.0

**Upgrade Tips:**
- The parameters of the `OverrideDynamicHitMasses` processor callback have changed. Keep in mind that you'll now be working with "inverse masses" (1 / mass) instead of masses. The implemented interface function in your processor should now look like this:
```cs
public void OverrideDynamicHitMasses(
    ref PhysicsMass characterMass,
    ref PhysicsMass otherMass,
    Entity characterEntity,
    Entity otherEntity,
    int otherRigidbodyIndex)
{
}
```
- The parameters of the `KinematicCharacterUtilities.DefaultMethods.UpdateGroundPushing` function have changed. In the standard characters, the function call should now look like this:
```cs
KinematicCharacterUtilities.DefaultMethods.UpdateGroundPushing(ref this, ref CharacterDeferredImpulsesBuffer, ref CharacterBody, DeltaTime, Entity, ThirdPersonCharacter.Gravity, Translation, Rotation, 1f);
```


**Changes:**
- update to Entities 0.51 and Unity 2021.3
- Added a "Limitations" page for the documentation website
- The processor's `OverrideDynamicHitMasses` now deals with `PhysicsMasses` instead of masses. This is because working with "inverse masses" makes it easier to represent infinite mass ratios, and it prevents a divide-by-0 pitfall. `PhysicsMass` also lets you change moment of inertia (the "rotational mass").
- Removed the `otherEntityIsCharacter` parameter of the processor's `OverrideDynamicHitMasses`. `OverrideDynamicHitMasses` cannot work when two dynamic characters are trying to push each-other, and so the function will simply not be called in that case
- In `KinematicCharacterBody`, `AddCloseHitsForVelocityProjection` was renamed to `ProjectVelocityOnInitialOverlaps` for more clarity
- `KinematicCharacterDeferredImpulse` now works with direct velocity changes rather than impulses
- Added a `KinematicCharacterUtilities.SetOrUpdateParentBody()` that takes care of properly setting parent data in the characterBody


**Fixes**
- Improved the correctness of rigidbody pushing when enabling `SimulateDynamicBody` on the character
- `KinematicCharacterUtilities.DefaultMethods.UpdateGroundPushing` now properly takes into account masses overriden by `OverrideDynamicHitMasses`
- Fixed jitter when a kinematic character was pushing a dynamic rigidbody
- fixed velocity projection issue that could result in our character staying stuck on the edge of a rolling sphere 


**Known Issues:**
- The [HideInInspector] attribute no longer seems to work, meaning some fields in authoring components that are meant to be hidden will now appear in the inspector. For example: the `FirstPersonPlayerAuthoring` component has a visible `Last Inputs Processing Tick` field that should normally not be visible
- [Platformer Sample] The animation for "standing up from a ledge grab" is not properly implemented. The real implementation would require root motion, so it will be done at a later date when the DOTS Animation package is availabe once again and offers support for root motion
- [OnlineFPS Sample] The game is capped out at 100 fps for the time being. Otherwise, if the framerate becomes much higher than the tickrate of 60, there is [a spam of warnings in editor and in build that severely affect the framerate](https://forum.unity.com/threads/server-world-allocations-live-too-long-in-0-50-preview-29.1254225/) . This appears to be an issue with the Netcode package that should be solved in future versions
- [OnlineFPS Sample] Player input will fail to be processed on some frames, once in a while. This might be the cause of remote player rotations looking a bit jittery, and local player rotation feeling slightly off. It can also result in shoot/jump inputs failing to be processed sometimes. This appears to be an issue related to [remote player prediction in the Netcode package](https://forum.unity.com/threads/commands-issue-when-trying-to-use-remote-players-prediction.1259031/#post-7999692), but more investigation is needed in order to confirm


## v0.4.1

**Upgrade Tips:**
- If you use code that sets the character's `ParentEntity`, you must now also remember to set a `ParentAnchorPoint` (in `KinematicCharacterBody`)


**Changes:**
- update to Entities 0.50.1-preview.2 and Physics 0.50.0-preview 43
- Added a `ParentAnchorPoint` field to `KinematicCharacterBody`


**Fixes**
- Fixed issues related to having character standing on small dynamic spheres, especially if the character has an elongated horizontal collider shape
- Fixed selection of best ground hit when there are multiple candidates, in order to prevent character from applying ground pushing to non-grounded overlapping rigidbodies
- Fixed computation of momentum added by leaving a moving parent object



## v0.4

**Upgrade Tips:**
- The changes in this version are pretty significant and specific. For this reason, if you need to upgrade, you should consider perhaps using the updated standard characters as a starting point and re-copying your current character's implementation into them. It's not very ideal, but it could end up being less trouble than the alternative of trying to manually upgrade your existing character. Regardless, the necessary upgrade steps are described below
- All upgrade steps for DOTS 0.50 in general will apply to your project. [Please refer to this page for details](https://docs.unity3d.com/Packages/com.unity.entities@0.50/manual/upgrade-guide.html). The main points are: 
    - All systems must have the `partial` keyword
    - Systems that deal with physics should use the new functions for handling dependencies (`RegisterPhysicsRuntimeSystemReadWrite()`) in `OnStartRunning()`, instead of handling these manually
    - Remove the `ENABLE_HYBRID_RENDERER_V2` scripting defineeeeeee
- In order to fix a potential case of non-determinism, all Rival characters now have a new `StoredKinematicCharacterBodyProperties` component on them (automatically added through the `KinematicCharacterUtilities.HandleConversionForCharacter()` function). That component is used in some `KinematicCharacterUtilities` functions, so you must add a `[ReadOnly] public ComponentDataFromEntity<StoredKinematicCharacterBodyProperties>  StoredKinematicCharacterBodyPropertiesFromEntity;` to your existing character update jobs, and then pass it to your character processor, and plug it in the right places (let the error messages guide you). You can look for all places where `StoredKinematicCharacterBodyProperties` is used in the standard characters, and use that as a reference. This replaces what used to be a `ComponentDataFromEntity<KinematicCharacterBody>` in the character System and Processor
- A `StoreKinematicCharacterBodyPropertiesSystem` must now update directly before all Rival characters. This will work automatically if you use the `KinematicCharacterUpdateGroup` for your character update system, but if you're not using it, you might have to copy that system and add it to your own update loop before your character systems
- In the character body inspector, character interpolation will now be set to interpolate only translation by default; not rotation. You might need to activate rotation interpolation if you were handling rotation in a fixed step job 


**Changes:**
- Upgraded DOTS packages (Entities, Physics, Hybrid Renderer, Netcode) to 0.50, and removed DOTS Animation package
- The `CharacterInterpolation` component now has separate options for interpolating translation and rotation, which means there are now two separate checkboxes in the character body inspector for interpolation. By default, only translation interpolation is active, since the rotation of standard characters is updated at a variable rate now
- Physics dependency handling for character systems now use the new `RegisterPhysicsRuntimeSystemReadWrite()` approach
- Added a `FixedUpdateTickSystem` to the standard characters packages, to help deal with the complexities of input handling processed on fixed update. This system is used in the "PlayerSystems" of both standard characters
- Added a new `StoredKinematicCharacterBodyProperties` component to all Rival characters by default, along with a `StoreKinematicCharacterBodyPropertiesSystem`. This change was made to fix a potential case of non-determinism in Rival characters, and it will affect all existing characters. See "Upgrade Tips" section above for more info. In the character job, a `ComponentDataFromEntity<StoredKinematicCharacterBodyProperties>` now replaces what used to be a `ComponentDataFromEntity<KinematicCharacterBody>`
- Added a new [Standard Character Overview](./Tutorial/tutorial-standardcharacterexplanation) section to the Tutorial, where we explain in detail how the Standard Characters work
- [Standard Characters] Rotation of all standard characters (FirstPerson and ThirdPerson) is now handled in a separate variable-rate system
- [Platformer Sample] Replaced the now-incompatible DOTS animation package with a hybrid animation approach, using Mecanim. The DOTS Animation package [is still expected to be released one day](https://forum.unity.com/threads/dots-development-status-and-next-milestones-march-2022.1253355/page-2#post-8000859), but it will have to wait until after DOTS 1.0
- [OnlineFPS Sample] Large parts of the code were refactored in order to improve quality, clarity and correctness. Some changes were made as requirements for Netcode 0.50, while others were made simply to provide a better example of how to handle prediction


**Fixes:**
- Fixed a potential case of non-determinism in the standard characters, which could happen during character-to-character collisions. The non-determinism was due to reading character velocity of other characters in the same job that modifies the velocity of a character. The switch to using a `ComponentDataFromEntity<StoredKinematicCharacterBodyProperties>` is due to this


**Known Issues:**
- The [HideInInspector] attribute no longer seems to work, meaning some fields in authoring components that are meant to be hidden will now appear in the inspector. For example: the `FirstPersonPlayerAuthoring` component has a visible `Last Inputs Processing Tick` field that should normally not be visible
- [Samples] Environment reflections do not work. [This might be an issue with the hybrid renderer package](https://forum.unity.com/threads/environment-reflections-not-working-with-hybrid-renderer-0-50-0.1256121/)
- [Platformer Sample] The animation for "standing up from a ledge grab" is not properly implemented. The real implementation would require root motion, so it will be done at a later date when the DOTS Animation package is availabe once again and offers support for root motion
- [OnlineFPS Sample] The game is capped out at 110 fps for the time being. Otherwise, if the framerate becomes much higher than the tickrate of 60, there is [a spam of warnings in editor and in build that severely affect the framerate](https://forum.unity.com/threads/server-world-allocations-live-too-long-in-0-50-preview-29.1254225/) . This appears to be an issue with the Netcode package that should be solved in future versions
- [OnlineFPS Sample] Player input will fail to be processed on some frames, once in a while. This might be the cause of remote player rotations looking a bit jittery, and local player rotation feeling slightly off. It can also result in shoot/jump inputs failing to be processed sometimes. This appears to be an issue related to [remote player prediction in the Netcode package](https://forum.unity.com/threads/commands-issue-when-trying-to-use-remote-players-prediction.1259031/#post-7999692), but more investigation is needed in order to confirm


## v0.3
**Upgrade Tips:**
-  As part of a fix for the implementation of `PreventGroundingWhenMovingTowardsNoGrounding` in standard characters, the call to `KinematicCharacterUtilities.DefaultMethods.DetectFutureSlopeChange()` has been **removed** from `characterProcessor.IsGroundedOnHit()`, and **moved** to `characterProcessor.OnUpdate()`. If you already had an existing character before upgrading, you may want to apply this fix to your character as well. Consult [this page](./Changelog/UpgradeTips/changelog-0.3-detectfutureslopeupgrade.md) for details


**Changes:**
- Removed the "Character Controller Wizard" window that generated character scripts from a template. This was done in order to focus on the "Standard Characters" packages as the main way to get started with Rival. Due to the differences between first-person and third-person characters, and all the particularities involved in their authoring/prefab setup, the standard characters are a safer and easier way to get started. This change also allows the learning resources to be more focused on only one approach. (Do note that characters that were previously generated with the "Character Controller Wizard" are still valid even though the window has been removed)
- Improved usability of FirstPerson StandardCharacter with a "FirstPersonCharacterUtilities" class that contains various methods for computing rotations for first person characters. The code of the FirstPerson character systems has been updated to use these methods
- Separated Standard Character packages in two distinct packages: FirstPerson and ThirdPerson
- Added default internal buffer capacities to various character DynamicBuffers 
- Changed the way we handle the framerate-independence of look input in Samples and Standard Characters. The look input is now expected to already be a delta for this frame, so it doesn't get multiplied by deltaTime when applied. This has been done due to the fact that raw mouse input is already a mouse delta for the frame in Unity's input system. Joystick look input, on the other hand, is not multiplied by deltaTime
- Changed the required parameters for PhysicsUtilities.GetHitFaceNormal()
- [Documentation] Made significant changes to the "Getting Started", "Core", "Tutorial", and "Standard Characters" sections of the documentation following the shift from the "Character Controller Wizard" window to the Standard Characters
- [Platformer Sample] Made character orient itself towards rope direction when in RopeSwing state
- [OnlineFPS Sample] Made general cleanup and improvements to the OnlineFPS sample code overall, especially regarding the handling of networked railgun VFX spawning


**Fixes:**
- Fixed an issue where the `PreventGroundingWhenMovingTowardsNoGrounding` option of standard & sample characters caused velocity stoppage in certain scenarios where we were moving near ledges.
- Fixed potential bug when a moving parent would make a character standing on it not collide with obstacles properly
- Fixed a faulty condition check for whether or not the character was overlapping with anything at the beginning of its update
- Fixed lerp calls where the ratio parameter was not clamped
- Made the SamplesBootstrap ignore all sample systems when not in a sample scene
- [Platformer Sample] Fixed issue where a character would always orient itself towards world forward when standing idle on water surface
- [Platformer Sample] Fixed a leak in the SimpleAnimation system
- [Platformer Sample] Fixed issue with skewed orientation of character while climbing
- [Platformer Sample] Fixed jumpy camera transitions when alternating rapidly between crouched and uncrouched states


**Known Issues:**
- Step Handling can cause errors when Burst compilation is disabled. This issue will require a future Unity.Physics package upgrade before it can be solved.
- [OnlineFPS Sample] In Unity 2020.3.16 and over, the console outputs errors during play mode for the OnlineFPS sample. However, this is only in the editor; not in builds. This issue is caused by an incompatilibity between the Unity.Netcode package and Unity 2020.3.16+ versions
- [OnlineFPS Sample] Jitter is visible on remote client characters. This issue will require a future Unity.NetCode package upgrade before it can be solved.
- [OnlineFPS Sample] Errors pop up in the console after compilation: "Couldn't find the TypeDescriptor for the type (...)". However, these errors are inoffensive and can be ignored. This issue will require a future Unity.NetCode package upgrade before it can be solved.
- [OnlineFPS Sample] Lightmaps do not work in builds unless the "OnlineFPS" scene is opened in editor while building. This issue will require a future package upgrade before it can be solved.
- [Platformer Sample] The "standing up from ledge hanging" animation is skipped entirely due to root motion issues. This issue will require a future Unity.Animation package upgrade before it can be solved.


## v0.2
**Fixes:**
- Fixed an invalid character present in the Samples project's files, causing compilation errors on certain machines
- Fixed unassigned spawn point in Basic sample

**Known Issues:**
- Step Handling can cause errors when Burst compilation is disabled. This issue will require a future Unity.Physics package upgrade before it can be solved.
- [OnlineFPS Sample] Jitter is visible on remote client characters. This issue will require a future Unity.NetCode package upgrade before it can be solved.
- [OnlineFPS Sample] Errors pop up in the console after compilation: "Couldn't find the TypeDescriptor for the type (...)". However, these errors are inoffensive and can be ignored. This issue will require a future Unity.NetCode package upgrade before it can be solved.
- [OnlineFPS Sample] Lightmaps do not work in builds unless the "OnlineFPS" scene is opened in editor while building. This issue will require a future package upgrade before it can be solved.
- [Platformer Sample] The "standing up from ledge hanging" animation is skipped entirely due to root motion issues. This issue will require a future Unity.Animation package upgrade before it can be solved.


## v0.1
**Features:**
- Initial package

**Known Issues:**
- Step Handling can cause errors when Burst compilation is disabled. This issue will require a future Unity.Physics package upgrade before it can be solved.
- [OnlineFPS Sample] Jitter is visible on remote client characters. This issue will require a future Unity.NetCode package upgrade before it can be solved.
- [OnlineFPS Sample] Errors pop up in the console after compilation: "Couldn't find the TypeDescriptor for the type (...)". However, these errors are inoffensive and can be ignored. This issue will require a future Unity.NetCode package upgrade before it can be solved.
- [OnlineFPS Sample] Lightmaps do not work in builds unless the "OnlineFPS" scene is opened in editor while building. This issue will require a future package upgrade before it can be solved.
- [Platformer Sample] The "standing up from ledge hanging" animation is skipped entirely due to root motion issues. This issue will require a future Unity.Animation package upgrade before it can be solved.