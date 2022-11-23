
# Character Hits

See [Tutorial - Character Hits](../Tutorial/tutorial-characterhits.md)

The character detects various hits during its update. These hits can be a result of ground detection, movement sweeps, or overlap resolution.

In order to access those hits, every character has a `DynamicBuffer<KinematicCharacterHit>` buffer containing all hits detected for this frame. Similarly, every character also has a `DynamicBuffer<StatefulKinematicCharacterHit>` for stateful hits (hits with an "Enter/Exit/Stay" state). However, those hits will only be processed if `KinematicCharacterProperties.ProcessStatefulCharacterHits` is enabled.

Make sure you only iterate on hits after all update steps of the `KinematicCharacterAspect` have bee called, otherwise you'll be at risk of missing some hits.