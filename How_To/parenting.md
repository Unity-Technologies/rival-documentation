
# Parenting the Character to Another Object

You are not allowed to make the Character object a child of another transform in the usual way (with the `Parent` component). However, you can use `KinematicCharacterAspect.SetOrUpdateParentBody` in order to set a parent entity for a character. However, you must remember to call this on every frame where you want the parenting behaviour to be active. This is because the common use case for character parenting is moving platforms, and in those cases, the "anchor point" of the parenting will change from frame to frame based on where the character collider makes contact with the ground. This function automatically handles storing that "anchor point" in the `KinematicCharacterBody` component.

`KinematicCharacterAspect.Update_MovingPlatformDetection` demonstrates how to do this.

Note 1: if you want to manually set the character's parent, you will have to remove the `KinematicCharacterAspect.Update_MovingPlatformDetection` update step of your character. Moving platform detection automatically assigns any valid moving ground entity as the current `KinematicCharacterBody.ParentEntity`, or assigns `Entity.Null` if no valid ground is detected. So keeping it in your character update would overwrite any parent you try to manually assign.