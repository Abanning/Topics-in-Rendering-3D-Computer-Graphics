### MP4: Multi-Light Illumination Infrastructure (Ambient and Diffuse)
# Overview:
This project builds the foundation of a real-time illumination system in a custom Unity shader. A single-pass shader computes per-pixel ambient and diffuse lighting from up to four simultaneous light sources, each configurable as a point light or a directional light, and applies the result to a textured surface. Light, material, and global rendering parameters are sent to the shader every frame from C# scripts and can be adjusted through a runtime UI. Key systems include:  

Illumination Model: For each pixel, the fragment shader computes a lighting term equal to an ambient contribution (`Ka` times the ambient color) plus a sum over the four lights of `Kd * max(N . L, 0)` times each light's color and intensity. The lighting term is then multiplied by the surface albedo, which is the sampled texture multiplied by the material color. Because the ambient term is multiplied by the albedo, the texture remains visible even under strong ambient light.  

Vertex Shader and Normals: The vertex shader transforms each vertex from object space to world space, view space, and clip space with explicit `mul()` calls. It passes the world-space position to the fragment shader so that lighting is evaluated per pixel in world space. Normals are transformed with the inverse transpose of the object-to-world matrix and normalized, which keeps them perpendicular to the surface under non-uniform scaling.  

Point and Directional Lights: The light direction `L` depends on the light type. For a point light, `L` is the normalized vector from the shaded point to the light's world position. For a directional light, `L` is the normalized negation of the light's forward vector, so the lighting direction is independent of the shaded position. The light type is encoded in the w component of each light's position/direction vector (1 for a point, 0 for a directional light), so a single vector array carries both kinds of data.  

Light Source Components: Each light is an empty GameObject with a LightSource component that exposes the light type, color, intensity, and an On/Off toggle. Three sliders (x, y, z) act as a position for point lights and as Euler rotation angles for directional lights, and switching between the two types resets the values appropriately. Point lights display a sphere whose size and color follow the light's intensity and color, rendered with a separate unlit shader and its own material instance. Directional lights have no visual marker and are controlled through rotation alone.  

Scene Control and Global Parameters: A SceneControl script gathers the state of up to four light sources every frame and uploads it to the shader as global arrays of positions/directions, colors (scaled by intensity), and enabled flags. Lights that are turned off are zeroed out. The same script packs the global Texture On/Off, Ambient On/Off, and Diffuse On/Off switches into a bit-mask that the shader decodes with bitwise AND operations, and it provides a Reset Scene option.  

Per-Object Materials: A MaterialLoader component gives each object its own material instance through `Renderer.material` and updates the ambient coefficient (`Ka`), ambient color, and diffuse coefficient (`Kd`) every frame. This allows every object in the scene to be tuned independently while sharing the same shader.  

# Screenshots:
<figure>
  <img width="600" height="335" alt="Material_Controls" src="https://github.com/user-attachments/assets/c57659f4-0764-4f3b-a326-4a64a6ddfb93" />
  <br />
  <figcaption>Each object's material properties can be modified independently. Ka is the ambient coefficient, Kd is the diffuse coefficient, and the RGB sliders define the ambient color; these parameters are used in the Phong Illumination equation.</figcaption>
</figure>
<br />
<br />

<figure>
  <img width="472" height="263" alt="Light_Controls" src="https://github.com/user-attachments/assets/a9d63761-8cfd-4c56-9715-417346da2de0" />
  <br />
  <figcaption>Each light source can be controlled independently. Users can choose a point light or directional light. The XYZ sliders control the translation (in point mode) or rotation (in directional mode). Each light can be turned on/off and the light intensity is a coefficient between 0 and 1.</figcaption>
</figure>
<br />
<br />

<figure>
  <img width="600" height="335" alt="DirectionalLight_Rotation" src="https://github.com/user-attachments/assets/d1eea427-3cb8-4f3e-ac57-113dd30759d2" />
  <br />
  <figcaption>Demonstrating directional light rotation sliders.</figcaption>
</figure>
<br />
<br />

<figure>
  <img width="471" height="263" alt="Global_SceneControl" src="https://github.com/user-attachments/assets/ebd3fd07-f91c-47e5-adef-7db669c5675b" />
  <br />
  <figcaption>Demonstrating global scene controls for texture, ambient light, and diffused light.</figcaption>
</figure>
<br />
<br />

# Code Release Restrictions:
Because this is a University of Washington Bothell course, I cannot publicly release my code solutions to these problems. However, code samples may be sent by request for hiring purposes (e.g. reviewing my previous work as it relates to a software engineering job).
