
# Input Handling

There are challenges related to handling input that has to be processed at a fixed timestep, as is the case with characters that update at a fixed rate.


## Intro to fixed update

Before we continue, we must make sure to understand how fixed updates work. The ECS comes with a `FixedStepSimulationSystemGroup` that acts as the default group where systems should be updated at a fixed rate.

On every regular update, we keep track of how long it's been since the last time we ran a fixed update. Then, we run the amount of fixed updates that corresponds to how many of them should've happened in that time (although in practice this is capped to a maximum amount of fixed updates per frame). Fixed updates happen at a fixed timestep instead of a variable one, and this makes them ideal for logic that needs consistency and predictability, like physics or server game simulation.

So if we have a fixed timestep of `0.02` seconds for example (meaning 50 fps), we're saying we want exactly 50 fixed updates to happen every second no matter what our framerate is. If we have a very high framerate and most frames take less than `0.02` seconds, then we'll have many frames where no fixed updates will happen, because the amount of time since the last fixed update will be less than `0.02` seconds. On the other hand, if we have a low framerate where some frames take `0.05` seconds, then multiple fixed updates will be triggered during these frames in order to "catch up" with the amount of fixed updates that should've happened in that `0.05` seconds time + the leftover time since the last fixed update on the previous frame.

In short: on any regular frame, you have to assume that fixed updates may happen zero times, one time, or many times.


## The problem with fixed timestep input

When you have systems updating at a fixed timestep that can consume punctual input events (like button presses/releases), two different things could go wrong:
1. If the framerate is higher than the fixed rate, you may get multiple regular updates between two fixed updates. That means your button press event could be set to true on frame1, then will be set back to false on frame2 because the button press didn't happen exactly on this new frame, and only then will you get a fixed update where the input value will be queried by a fixed-rate system (false). So even though you pressed the button, that input event was lost between two fixed updates, and the press event will never be processed.
1. If the framerate is lower than the fixed rate, you may get multiple fixed updates between two regular updates. That means your button press event might be set to true on frame1, then a system will read that value (true) on fixedframe1 and process that input, and then the same system will update again on fixedframe2 and process that input once again because it is still true (the input system didn't get a chance to set it back to false, because it only updates the value on regular updates). So you pressed the input once, but the event got processed multiple times.

Here's what these problems mean in practice, using a jump button press as an example, with a character system handling jump on fixed update:
1. Your framerate is higher than the fixed rate. You press the jump button, but the character doesn't jump. That's because the jump press event was lost between two fixed updates.
1. Your framerate is lower than the fixed rate. You press the jump button, and your character jumps much higher than it normally should. After some degugging, you understand that this is because your character supports double-jumps, and the jump button press was processed by your character system on two fixed updates in a row. So one single jump button press lead to a double-jump.

Changing the core input system in order to make it update only on fixed update wouldn't be an ideal solution, because it's highly likely that one game might need some input to be processed at a variable rate (camera rotation input) while other input should be processed at a fixed rate (character jump input). So we need to find a way to solve this while still having our raw input queried at a variable rate as usual.

Moreover, if we chose to solve these problems by setting a boolean to true only when the input is pressed, and then waiting for a fixed-rate system to consume that input and set it back to false, we'd have a potentially important limitation: there can't be multiple consumers of that input event. Or maybe there could be multiple consumers, but we'd have to remember to manually set input values back to false at the end of each fixed update for each individual input event. This solution also wouldn't be a great fit for netcode, because it would incur additional state data to synchronize for inputs. Overall this solution would work in certain cases, but we can do better.


## A solution for fixed timestep input

When processing input events at a fixed update, there are two things to keep in mind in order to avoid problems:
1. We must remember if the input event happened at any point since the last fixed update; not just if it happened on this frame. This solves the problem of multiple regular updates happening between fixed updates.
1. We must only process input events on the first fixed update where they have been detected; not on subsequent ones. This solves the problem of multiple fixed updates between regular updates.

Player input handling in the standard characters is structured in a way that solves all of these problems:
* Button press/release events are handled by a `FixedInputEvent` struct. 
* When an input event happens in regular update, we call `FixedInputEvent.Set(tick)`. When querying whether or not we should process an input event in a fixed update, we call `FixedInputEvent.IsSet(tick)`
* The "tick" parameter is a counter counting how many fixed updates happened so far. It is updated by `FixedTickSystem` at the end of the `FixedStepSimulationSystemGroup`
* By using this "tick", the `FixedInputEvent` can determine if an input event happened at any point since the last fixed update, and if the input event truly happened on this fixed update or a previous one.

You should remember these rules when trying to add new button press/release events to your character. You can refer to the standard characters implementation of the `JumpPressed` input if you want to see how this works in practice.
