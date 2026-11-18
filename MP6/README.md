### MP6: Projected Planar Shadows with Stencil Masking and Alpha Blending
# Overview:
This project adds shadows to the Blinn-Phong illumination system from the previous assignment by projecting object geometry onto a planar receiver. Rather than using a shadow map, a multi-pass shader uses line-plane intersection to flatten each shadow-casting mesh onto the ground plane, then uses the stencil buffer and alpha blending to draw the flattened geometry as a darkening shadow. Directional, point, and spot lights each cast their own shadows, and the shadows from point and spot lights fade with distance and with the spot cone. Key systems include:  

Shadow Projection: Each vertex of a shadow caster is moved along a line onto the receiver plane. For a point on the object, `V`, and a ray direction, `d`, the intersection with the plane `(n . p) - D = 0` is found by solving for the distance along the ray, `t = (D - n . V) / (n . d)`, and the shadow vertex is `V + t * d`. A directional light uses one shared ray direction for every vertex, so its shadow has parallel rays. A point or spot light uses the ray from the light's position through each vertex. If a ray is nearly parallel to the plane, the denominator approaches zero, and the vertex is collapsed instead of being projected. The plane normal and offset are computed each frame from the receiver object's transform, so the receiver can be moved or rotated.  

Multi-Pass Shader: Each shadow caster uses a four-pass shader: one shadow-projection pass each for the directional, point, and spot lights, followed by the Blinn-Phong illumination pass that draws the object itself. The receiver uses a separate shader that applies the same Blinn-Phong illumination model and also marks its pixels in the stencil buffer. The shadow passes are drawn after the receiver in the render queue.  

Stencil Masking and Blending: The receiver writes a reference value of 1 into the stencil buffer for every pixel it covers. The shadow-projection passes only draw where the stencil value equals 1, so shadows appear only on the receiver and never spill onto the background or onto other geometry. The shadow passes disable depth writes and use standard alpha blending, so the shadow color is mixed into the existing color buffer. Because each light has its own pass, overlapping shadows from different lights blend on top of each other and darken the surface cumulatively.  

Z-Fighting Avoidance: Shadow geometry lies on the same plane as the receiver, which would cause depth fighting. Each shadow pass offsets its projected vertices slightly along the plane normal, using a slightly larger offset for each successive pass, so that the passes resolve cleanly in the depth buffer.  

Distance and Angular Attenuation: Directional shadows have a constant strength. Point and spot shadows fade linearly between the light's near and far distances, measured from the light to the shadow position on the plane, and disappear completely beyond the far distance. Spot shadows also use the light's cone: the inner and outer cone angles are converted to half-angles, the shadow has full strength inside the inner cone, falls off with a `smoothstep` between the inner and outer cones, and does not exist outside the outer cone. As a result, shadow edges fade gradually instead of ending abruptly.  

Shadow Controls: Each shadow-casting light type has a user-defined shadow color and shadow strength. A Shadow Manager component packs three On/Off flags (directional, point, and spot) into a bit-mask that the shadow fragment programs decode with bitwise AND operations. A shadow type is automatically disabled while a light of that type is switched off, and its toggle is shown as unavailable in the UI until the light is back on.  

Light and Receiver Data: Small loader scripts attached to each light send the receiver plane, the light position or direction, the cone angles, the near and far distances, and the shadow color and strength to the shader as global properties every frame. As a result, moving or rotating a light immediately changes the shape and position of its shadow.  

Runtime UI: A reflection-based control panel generates sliders, toggles, color controls, and buttons from the public fields of every component that implements a simple interface. Light objects also receive position and rotation sliders, which makes it possible to move and aim the lights and watch the shadows respond. The panel can be dragged, resized, scaled, and shown or hidden with a key.  

# Screenshots:
<figure>
  <img width="600" height="335" alt="Adjust_UI" src="https://github.com/user-attachments/assets/2bae7bef-fe89-4f54-98c5-514719a32501" />
  <br />
  <figcaption>If the UI is too small, grab the bottom right corner and drag to extend, then adjust the scale slider on top to make the text inside bigger.</figcaption>
</figure>
<br />
<br />

<figure>
  <img width="600" height="335" alt="Shadow_Controls" src="https://github.com/user-attachments/assets/f51078c9-0203-4631-b8d0-0383f14517c1" />
  <br />
  <figcaption>You can independently control which light sources produce shadows with the global shadow manager.</figcaption>
</figure>
<br />
<br />

<figure>
  <img width="448" height="250" alt="Directional_Shadow" src="https://github.com/user-attachments/assets/111116cb-fd49-4a88-b103-a69449674b9e" />
  <br />
  <figcaption>Directional light source 1 produces dynamically projected shadows from vector math. Rotating the first directional light source controls the position of the shadows on the ground plane.</figcaption>
</figure>
<br />
<br />

<figure>
  <img width="600" height="335" alt="Point_Shadow" src="https://github.com/user-attachments/assets/461e6426-e329-4101-ad31-2c4ae1815761" />
  <br />
  <figcaption>Point light shadows move when the light moves.</figcaption>
</figure>
<br />
<br />

<figure>
  <img width="600" height="335" alt="Penumbra" src="https://github.com/user-attachments/assets/fd2ae7e7-fbef-4dc1-92cd-65b225efccec" />
  <br />
  <figcaption>Adjusting the near distance within the far distance bound causes a blending effect between the near and far distance intensities.</figcaption>
</figure>
<br />
<br />

<figure>
  <img width="600" height="335" alt="Light_Shadow_Blend" src="https://github.com/user-attachments/assets/265a42ab-d193-4f71-a627-690495acd167" />
  <br />
  <figcaption>Adjusting light intensity affects the light color and shadow color blending based on distance, angle, and the Blinn-Phong Illumination equation.</figcaption>
</figure>
<br />
<br />

<figure>
  <img width="440" height="246" alt="Spot_Rotation" src="https://github.com/user-attachments/assets/1a8a1657-c281-4e7c-bc2a-7d0d543a0107" />
  <br />
  <figcaption>Similar to the directional light, shadows respect the spot light rotation.</figcaption>
</figure>
<br />
<br />

<figure>
  <img width="369" height="206" alt="Spot_Penumbra" src="https://github.com/user-attachments/assets/479f652e-7846-45d5-b028-b78b3afad9b3" />
  <br />
  <figcaption>Spot lights respect distance and angular attenuation. Near distance, far distance, and dropoff control the distance attenuation of the light, and the cone angles and angular dropoff control the angular attenuation. Notice how the edges become more clear/more fuzzy as changes are made.</figcaption>
</figure>
<br />
<br />

# Code Release Restrictions:
Because this is a University of Washington Bothell course, I cannot publicly release my code solutions to these problems. However, code samples may be sent by request for hiring purposes (e.g. reviewing my previous work as it relates to a software engineering job).

