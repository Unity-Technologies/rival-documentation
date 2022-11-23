
# First Person Standard Character Setup

1. Import the `StandardCharacter_FirstPerson.unitypackage` into your project by double-clicking on it. It is under "Rival/StandardCharacters". [[Screenshot]](../../Images/std_firstperson_1.PNG)
1. Drag and drop the `FirstPersonCharacter` and `FirstPersonPlayer` prefabs into a subscene. These prefabs are under "Rival_StandardCharacters/FirstPerson/Prefabs". [[Screenshot]](../../Images/std_firstperson_2.PNG)
1. In the subscene, assign the `FirstPersonCharacter` GameObject as the "Controlled Character" in the `FirstPersonPlayer` GameObject's `FirstPersonPlayerAuthoring` component. [[Screenshot]](../../Images/std_firstperson_3.PNG)
1. Find the `View` GameObject under in the `FirstPersonCharacter` GameObject's hierarchy, and add a `MainEntityCameraAuthoring` component on it. This component marks that entity as the entity that your GameObject camera must follow. [[Screenshot]](../../Images/std_firstperson_4.PNG)
1. Make sure your scene has a camera GameObject that is *not* in a subscene (cameras are not bakeable to entities yet), and add a `MainGameObjectCameraAuthoring` component on it. This component marks that camera as the GameObject that must copy the entity marked with `MainEntityCameraAuthoring` every frame. [[Screenshot]](../../Images/std_firstperson_5.PNG)


### Notes
* The `View` GameObject represents the "camera point" of the first person character. When controlling the look input of the character, this `View` entity is what's being rotated up and down