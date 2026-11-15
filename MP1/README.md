### MP1: Custom Shaders and Runtime Color Control
# Overview:
This project reintroduces the Unity workflow of scripts, materials, and shaders, and establishes the basic pipeline for sending data from CPU-side C# code to a custom GPU shader at runtime. Scene objects are rendered with a single hand-written shader, `552_Shader.shader` (listed under the `552_Shaders` category), and an on-screen Color Editor lets the user select an object and change its color interactively. Key systems include:  

Custom Shader: The shader declares two properties, a main texture (`_MainTex`) and a color (`_Color`). The vertex program (`VertexProgram`) transforms each vertex from object space to clip space with explicit `mul()` calls, applying the object-to-world matrix (`unity_ObjectToWorld`), the view matrix (`UNITY_MATRIX_V`), and the projection matrix (`UNITY_MATRIX_P`) in sequence, rather than using a built-in transform helper. The fragment program (`FragmentProgram`) outputs the sampled texture value multiplied by `_Color`, so the object's color is determined on the GPU by the value supplied from script.  

Object Selection: Left-clicking in the scene casts a ray from the main camera through the mouse position using `Physics.Raycast`, restricted to objects on the `Selectable` layer. Clicks that land on the UI are ignored so that adjusting a slider never changes the selection. Clicking an empty area deselects the current object, resets the sliders to zero, and disables them.  

RGB Color Editor: Three sliders (R, G, B) control the color of the selected object. When an object is selected, its current color is read from its renderer and used to initialize the sliders. Moving a slider updates the corresponding color channel and writes the new color to the selected object's material, which updates the `_Color` property consumed by the fragment shader. A label reports the name of the currently selected object, and a Reset button reloads the scene to restore the initial state.  

Per-Object Materials: All objects in the scene share the same custom shader and material setup, but each object is edited independently. Accessing `Renderer.material` at the time of the color change gives the selected object its own material instance, so changing the color of one object does not affect the others.  

# Screenshots:
<figure>
  <img width="600" height="336" alt="Color_Editing" src="https://github.com/user-attachments/assets/5596ee99-9e03-4a49-a83d-ea7adac424e5" />
  <br />
  <figcaption>Demonstrating our knowledge of basic Unity skills as an intro to this class and refresh on previous concepts like Raycast, matrix manipulation, and shader code.</figcaption>
</figure>
<br />


# Code Release Restrictions:
Because this is a University of Washington Bothell course, I cannot publicly release my code solutions to these problems. However, code samples may be sent by request for hiring purposes (e.g. reviewing my previous work as it relates to a software engineering job).
