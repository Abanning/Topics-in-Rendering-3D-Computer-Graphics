### MP5: Blinn-Phong Illumination with Spot Lights and Attenuation
# Overview:
This project completes a real-time Blinn-Phong illumination system in a custom Unity shader. Building on the multi-light ambient and diffuse infrastructure from the previous assignment, it adds a normalized specular term, spot lights, and both distance and angular light attenuation. A single-pass shader evaluates lighting per pixel for up to five light sources. Light, material, and global rendering parameters are uploaded from C# scripts every frame and can all be adjusted at runtime through a generated UI. Key systems include:  

Illumination Model: For each pixel, the fragment shader adds an ambient term (`Ka` times the ambient color) to a sum over all enabled lights. Each light contributes a diffuse term, `Kd * max(N . L, 0)` times the diffuse color, plus a normalized Blinn-Phong specular term, `Ks * (s + 2) / (2π) * (N . H)^s`, where `s` is the shininess exponent, `H` is the halfway vector between the light direction and the view direction, and the specular color is user-defined. Both terms are scaled by the light's color and intensity and by the light's combined attenuation. The total lighting is then multiplied by the surface albedo, which is the sampled texture multiplied by the base color. The vertex shader transforms vertices with explicit `mul()` calls, passes the world-space position to the fragment shader, and transforms normals by the inverse transpose of the object-to-world matrix.  

Light Types: Each light can be configured as a point light, a directional light, or a spot light. For point and spot lights, the light direction is the normalized vector from the shaded point to the light's world position. For directional lights, it is the negated forward vector of the light, so the direction does not depend on the shaded position. Position and orientation come from each light's Transform, so spot and directional lights are aimed by rotating the light object.  

Distance Attenuation: Point and spot lights use two user-defined distances, near and far. The light has full intensity at distances up to the near distance and no contribution beyond the far distance. Between the two, a `smoothstep`-based falloff based on `1 - (n * n) / (d * d)` reduces the intensity with distance. Directional lights are not attenuated by distance.  

Angular Attenuation: Spot lights use an inner cone angle, an outer cone angle, and a dropoff exponent. The shader compares the cosine of the angle between the spot direction and the direction from the light to the shaded point against the cosines of the two cone angles. Inside the inner cone the light has full intensity, outside the outer cone it contributes nothing, and in between the normalized cosine difference is raised to the dropoff exponent.  

View-Dependent Specular: The camera position is sent to the shader every frame, and the view vector is computed per pixel in world space. As the camera moves, the specular highlights shift across the surface.  

Light Source Components: Each light is a GameObject with a LightSource component that exposes the light type, color, intensity, an On/Off toggle, the cone angles and dropoff, and the near and far distances. Point and spot lights display a sphere whose size follows the light's intensity and whose color matches the light's color, and the sphere turns black when the light is off. Directional lights have no visual marker.  

Scene Control and Bit-Mask Encoding: A SceneControl script gathers the state of every light each frame and uploads it to the shader as global arrays of positions, directions, colors (scaled by intensity), cone angles, dropoffs, distances, light types, and enabled flags. Six global toggles (Texture, Ambient, Diffuse, Specular, Angular Attenuation, and Distance Attenuation) are packed into a single integer bit-mask that the shader decodes with bitwise AND operations. A Reset Scene option reloads the scene.  

Per-Object Materials: A MaterialLoader component gives each object its own material instance and updates the ambient, diffuse, and specular coefficients and colors, as well as the shininess exponent, every frame. This allows every object in the scene to be tuned independently while sharing the same shader.  

Runtime UI: A reflection-based UI manager discovers every component that implements a simple interface and builds a control panel from its public fields: toggles for booleans, buttons for enums such as the light type, sliders for numeric values (using each field's range), and RGB sliders for colors. Light objects also receive position and rotation sliders. The panel can be dragged, resized, and scaled.  

# Screenshots:
<figure>
  <img width="600" height="335" alt="Material_Properties" src="https://github.com/user-attachments/assets/48f1b90c-340f-40d4-976d-a29146b4e816" />
  <br />
  <figcaption>Each object has individual material properties like the previous project. Now, we have full control over ambient (Ka and ambient color), diffuse (Kd and diffuse color), and specularity (specular color and shininess) components of the Blinn-Phong Illumination model.</figcaption>
</figure>
<br />
<br />

<figure>
  <img width="446" height="249" alt="Spot_Light" src="https://github.com/user-attachments/assets/8f9bf5a8-8635-4bc0-ad63-f322d6fbae86" />
  <br />
  <figcaption>Light sources can now be configured in spot light mode, allowing rotation and translation to shine light on objects in view.</figcaption>
</figure>
<br />
<br />

<figure>
  <img width="471" height="263" alt="Global_SceneControl" src="https://github.com/user-attachments/assets/9ffb94cc-5aa1-482b-85f1-41e9bcaa6665" />
  <br />
  <figcaption>Demonstrating global scene controls</figcaption>
</figure>
<br />
<br />


# Code Release Restrictions:
Because this is a University of Washington Bothell course, I cannot publicly release my code solutions to these problems. However, code samples may be sent by request for hiring purposes (e.g. reviewing my previous work as it relates to a software engineering job).
