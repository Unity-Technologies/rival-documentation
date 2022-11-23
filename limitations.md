
# Limitations

## General

* The root character entity must always have a (1,1,1) scale. You are allowed to scale child entities though
* The root character entity must never be a child of another entity. However, there is a mecanism to [immitate parenting for characters](./How_To/parenting.md) 

## Dynamic Physics Interactions

Dynamic physics interactions (pushing and being pushed by dynamic rigidbodies) are an area where a kinematic character controller will have the most limitations, when compared to a character controller implemented fully with rigidbody physics. However, a dynamic rigidbody character controller would have plenty of different limitations of its own (limited control over movement and collision-solving), so the choice of kinematic VS dynamic characters is always a choice of trade-offs.

This character controller offers support for [dynamic rigidbody interactions](./How_To/dynamic-body-interaction.md), which tries to immitate the behaviour of a fully dynamic character. There are, however, some limitations:

* The character rigidbody's movement cannot be constrained by physics joints. This means you wouldn't be able to have a physics rope where one end is attached to the ground and the other end is attached to the character, and have that rope prevent the character from moving away. You could, however, have a chain of joints that's attached to the kinematic character rigidbody, and have that chain get moved along with the character. The important thing to remember here is that the character will always be a kinematic rigidbody, and so it will have the same limitations with joints that any regular kinematic rigidbody would have.
* While characters can simulate believable dynamic physics interactions with a single rigidbody, dynamic physics interactions between more than 2 bodies simultaneously will not be physically-accurate. For example, if we have a seesaw with a character standing on one side and a counter-weight on the other side; we have a dynamic interaction between 3 bodies. In this case, the "ground pushing" force of the character on the seesaw will not be aware of the existence of the counter-weight, and the result will be that the force it applies on the seesaw will be weaker than it should really be. So if both the character and the counter-weight have a mass of 1, and they are both at an equal distance from the seesaw's pivot, the seesaw will tilt towards the counter-weight instead of being in balance
* A character with `SimulateDynamicBody` set to false (kinematic character) will not be able to push a character with `SimulateDynamicBody` set to true (dynamic character). In order for one character to be able to push another, both of them must have `SimulateDynamicBody` set to true