
# Changelog

[Previous Versions](./Changelog/changelog-archive.md)

## v1.0.0-pre

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