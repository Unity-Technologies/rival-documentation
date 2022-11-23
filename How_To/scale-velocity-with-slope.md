
# Scale Velocity With Slope Angle

You may want to make your character move slower uphill, and faster downhill. Here's one way of doing that:

The `CharacterControlUtilities.GetSlopeAngleTowardsDirection` function can calculate the signed slope angle in a given movement direction. The resulting angle will be positive if the slope goes up, and negative if the slope goes down.

Now, in your character aspect's `HandleVelocityControl`, you can simply apply a multiplier to your desired character velocity based on that signed slope angle. 