### MP2: Multi-Pass Vertex Shaders and Object Space Geometry
# Overview:
This project focuses on modifying geometry in object space with a custom vertex shader. A procedurally generated grid mesh is drawn twice by a two-pass shader: the first pass renders the mesh as a textured plane, and the second pass redraws it as a displaced, sinusoidal "wave" surface above the original. All wave and color parameters are controlled in real time through a runtime UI. Key systems include:  

Procedural Mesh: A provided mesh component (MyMesh) builds an N x N grid of vertices on the X/Z plane, spanning [-1, 1] in object space at y = 0, with upward-facing normals, UV coordinates, and two triangles per grid cell. The resolution can be changed at runtime from 2x2 to 100x100 to trade off wave smoothness against vertex count.  

Two-Pass Shader: Both passes use the same mesh and texture. Pass 1 transforms each vertex from object space to world space, applies a 10x scale, and then transforms it through the view and projection matrices using explicit `mul()` calls; its fragment shader outputs the sampled texture. Pass 2 uses the same transform chain, but first displaces each vertex along the object-space Y axis. Back-face culling is disabled so both sides of the displaced surface are visible.  

Sinusoidal Displacement: In the second pass, each vertex's X and Z coordinates are remapped from [-1, 1] to [0, 1] and used to compute a sine or cosine wave along each axis. The Period parameter sets the number of full waves across the mesh, and the Amplitude parameter scales the wave height, with separate values for X and Z. The two wave offsets are linearly blended with a single XZ blend weight and added to a user-controlled Y offset, all in object space before the world transform and 10x scale are applied.  

Animation: Each axis can be animated independently. When enabled, a phase term built from Unity's built-in `_Time.x` and an integer speed multiplier (1 to 10 for each axis) is added to the wave angle, so the surface continuously travels across the mesh.  

Bit-Mask Flag Encoding: To limit the number of shader parameters, the boolean options are packed into a single integer bit-mask: bit 0 selects sine or cosine for X, bit 1 selects sine or cosine for Z, bit 3 enables the second pass, bit 4 enables X animation, and bit 5 enables Z animation. A script (FunctionEncode) encodes the mask each frame and sends it, along with all numeric and color parameters, to the shader as global shader properties. The shader decodes the mask with bitwise AND operations. When the second-pass bit is cleared, the second pass collapses every vertex to the origin and outputs a zero color, so the displaced surface is not visible.  

Fragment Color Blending: The second pass fragment shader linearly interpolates between a user-selected Base Color and the sampled texture color using a Color Weight in [0, 1], where 0 shows only the base color and 1 shows only the texture.  

Runtime UI: The shader and mesh settings are exposed as two panels ("Shader Controls" and "Mesh Controls") containing toggles and sliders, and the panel scale and width can also be adjusted. Interactive camera manipulation allows the surface to be examined from any angle.  

# Screenshots:
<figure>
  <img width="600" height="337" alt="UI_Scaling" src="https://github.com/user-attachments/assets/181b781e-5bab-4452-90e0-308ead909eb4" />
  <br />
  <figcaption>Demonstrates how to adjust the UI scale if it is too small to use. Select the second slider from the bottom and drag to the right until it becomes more visible. These projects initially were built to run inside of Unity, but to avoid requiring you to download and install Unity and release my project code publicly, I have come up with this dynamic UI solution.</figcaption>
</figure>
<br />

<figure>
  <img width="600" height="337" alt="Basic_Controls" src="https://github.com/user-attachments/assets/f3170bac-8928-4a6b-a1ca-655a765dd24c" />
  <br />
  <figcaption>Demonstrating basic UI controls on the second pass filter. Control the y (height) offset, the x and z amplitude and period, and other aspects of the runtime mesh.</figcaption>
</figure>
<br />

<figure>
  <img width="600" height="337" alt="Color_Controls" src="https://github.com/user-attachments/assets/04379e68-21a0-41ce-b5d1-8fcce66534a7" />
  <br />
  <figcaption>Demonstrating the color blending of the texture and user-selected color.</figcaption>
</figure>
<br />

<figure>
  <img width="600" height="337" alt="XZ_Blend" src="https://github.com/user-attachments/assets/3b277b33-8482-4a96-ac77-e89869307318" />
  <br />
  <figcaption>Demonstrating the blending between X- and Z-axis sinusoidal functions.</figcaption>
</figure>
<br />

<figure>
  <img width="600" height="337" alt="Animation" src="https://github.com/user-attachments/assets/211927d1-3026-4a99-ae9c-17fff3f465de" />
  <br />
  <figcaption>Demonstrating the animation effect achieved by using Sin(time) GPU functions.</figcaption>
</figure>
<br />

<figure>
  <img width="600" height="337" alt="Resolution_Control_Animated" src="https://github.com/user-attachments/assets/3b0f40a6-28d0-45f2-bce6-8b46defd68ff" />
  <br />
  <figcaption>Demonstrating the effect of changing the mesh resolution on the animation smoothness.</figcaption>
</figure>
<br />

# Code Release Restrictions:
Because this is a University of Washington Bothell course, I cannot publicly release my code solutions to these problems. However, code samples may be sent by request for hiring purposes (e.g. reviewing my previous work as it relates to a software engineering job).
