
# Jumping

Due to the character's grounding mecanisms, there are particularities to be aware of when it comes to jumping. Trying to make a character jump by simply giving it an upwards velocity will often fail because the character will still think it is grounded, and it'll try to snap right back to the ground on the following frame.

When trying to jump, it is required to manually unground the character by setting `KinematicCharacterBody.IsGrounded` to `false`. This will make sure that the ground snapping mechanism won't snap your character back to ground. You can also use the pre-made `CharacterControlUtilities.StandardJump` function for a quick complete jump implementation.


## Double-jumping, Triple-jumping, etc...

Double-jumping is simply a question of keeping track of how many jumps you've done in-air, and resetting that value whenever you are grounded. If you are in-air, you can allow your character to jump just as it would jump on ground, but only if the amount of in-air jumps made is lower than the maximum number of in-air jumps allowed (in this case, that number would be 1, but it could be higher if you want triple jumps, etc...).


## Jump higher when holding jump

Sometimes you want your character to jump higher when holding jump input than when quickly tapping jump. To do this, you would have the following variables in your character component: `JumpBaseForce`, `JumpHoldAcceleration`, and `JumpHoldDuration` (the names are only suggestions). When the jump input is first pressed, you can make your character jump regularly with the `JumpBaseForce`. But then, every consecutive frame where the jump input is still held and the time elapsed since the regular jump is lower than `JumpHoldDuration`, you will add an additional upward acceleration corresponding to `JumpHoldAcceleration` to the character's velocity.


## Jumping grace times

If your character is in-air and you want to make sure pressing jump slightly before the character lands on the ground will still result in a jump, you can do so by always remembering the time at which the jump input was pressed, even when you're in-air. When your character becomes grounded, you can check the last time jump was pressed, and then jump regularly if that time was within a certain grace duration.

If your character recently became ungrounded and you want to make sure pressing jump will still result in a jump even though the character isn't grounded anymore, you can always remember the last time the character was grounded. Then, when in-air and pressing jump, you can check the last time the character was grounded, and then jump regularly if that time was within a certain grace duration.