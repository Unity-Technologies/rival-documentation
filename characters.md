# Characters

## Overview

A character controller is an entity that has a physics shape, a kinematic physics body, and a handful of character components. Your main means of controlling the character is by setting its velocity through the `CharacterBody` component, and setting it rotation through the regular transform components.

The heart of the character implementation is in the `KinematicCharacterAspect`. This aspect operates on all the required components for core character logic, and contains methods that must be called by the user for updating the character. Many of these methods require a `IKinematicCharacterProcessor` to be passed as parameter. The role of the `IKinematicCharacterProcessor` is to give you an opportunity to customize some of the logic that happens during the internal character updates.

The recommended way of working with this package is to create your own custom character aspect that includes the `KinematicCharacterAspect` and implements `IKinematicCharacterProcessor`. That aspect can then pass itself as the `IKinematicCharacterProcessor` parameter to the `KinematicCharacterAspect` methods it calls for updating the character. With this setup, the `ICharacterProcessor` methods of your aspect can access all of the data of the character, because these methods will be part of the character aspect. The "Standard Characters" that come with this package use this kind of setup.

All of the required character components can be added to an entity by calling `KinematicCharacterUtilities.BakeCharacter` in a `Baker`. Once again, the "Standard Characters" demonstrate this.


## Details

- [Concepts](./Characters/characterconcepts.md)
- [Components](./Characters/charactercomponents.md)
- [Character Aspect](./Characters/characteraspect.md)
- [IKinematicCharacterProcessor](./Characters/icharacterprocessor.md)
- [Systems](./Characters/charactersystems.md)
- [Utilities](./Characters/characterutilities.md)
- [Standard Characters](./Characters/standardcharacters.md)