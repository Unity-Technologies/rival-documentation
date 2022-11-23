
# Third Person Standard Character Setup

1. Import the `StandardCharacter_ThirdPerson.unitypackage` into your project by double-clicking on it. It is under "Rival/StandardCharacters". [[Screenshot]](../../Images/std_thirdperson_1.PNG)
1. Drag and drop the `ThirdPersonCharacter`, `ThirdPersonPlayer`, and `OrbitCamera` prefabs into a subscene. These prefabs are under "Rival_StandardCharacters/ThirdPerson/Prefabs". [[Screenshot]](../../Images/std_thirdperson_2.PNG)
1. In the subscene, assign the `ThirdPersonCharacter` GameObject as the "Controlled Character" in the `ThirdPersonPlayer` GameObject's `ThirdPersonPlayerAuthoring` component. [[Screenshot]](../../Images/std_thirdperson_3.PNG)
1. In the subscene, assign the `OrbitCamera` GameObject as the "Controlled Camera" in the `ThirdPersonPlayer` GameObject's `ThirdPersonPlayerAuthoring` component. [[Screenshot]](../../Images/std_thirdperson_4.PNG)
1. Add a `MainEntityCameraAuthoring` component on the `OrbitCamera` GameObject. This component marks that entity as the entity that your GameObject camera must follow. [[Screenshot]](../../Images/std_thirdperson_5.PNG)
1. Make sure your scene has a camera GameObject that is *not* in a subscene (cameras are not bakeable to entities yet), and add a `MainGameObjectCameraAuthoring` component on it. This component marks that camera as the GameObject that must copy the entity marked with `MainEntityCameraAuthoring` every frame. [[Screenshot]](../../Images/std_thirdperson_6.PNG)