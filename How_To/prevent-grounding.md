
# Prevent Grounding on Certain Hits

Preventing grounding procedurally with code on certain hits can be done with the `IsGroundedOnHit` callback of your character aspect. In this callback, you could for example look at the presence of certain components on the hit entity, or look at the physics tags/categories of that object, and decide to `return false;` when you want the character to not be able to consider itself grounded on that hit