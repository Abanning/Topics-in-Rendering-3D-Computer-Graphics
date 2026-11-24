### MP8: Shadow Mapping and Planar Mirror Reflections
# Overview:
This project extends the Blinn-Phong renderer from the previous assignments with two multi-camera rendering techniques: real-time shadow mapping and planar mirror reflection. Each technique renders the scene from an additional camera into an off-screen render texture, and the results are combined with the main camera's image. Both work in the same scene, alongside the ground fog and torch post-processing effects from the previous assignment, and every parameter can be adjusted at runtime through an in-game control panel. Key systems include:  

Shadow Map Generation: A shadow camera is attached to Light 0 and renders the scene into a floating-point render texture using a replacement shader. For each pixel, the shader stores the world-space position of the visible surface in the RGB channels and its distance from the camera, and therefore from the light, in the alpha channel. The shadow map is therefore a map of radial distance to the light, not clip-space depth. Only objects tagged as depth-writing in their shader are drawn into the map, so the mirror plane and the light-source visuals neither cast shadows nor appear in it. The shadow camera uses a 120° field of view, and a preview quad in the scene can display the map for debugging.  

Shadow Lookup and Blending: In the vertex shader, each vertex is offset along its world-space normal by a normal bias and projected with the shadow camera's view-projection matrix to obtain shadow-map texture coordinates. In the fragment shader, the distance from the light to the fragment is compared against the stored distance plus a depth bias. The lit fraction is converted to a shadow factor using the shadow intensity, `lerp(1 - intensity, 1, litFraction)`. This factor weights the light's Blinn-Phong contribution against a user-selected shadow color, so shadows can be tinted instead of simply darkened. The factor fades back to fully lit within the outer 15% of the shadow map, which avoids hard edges where objects leave the shadow camera's view. Only Light 0 is shadowed, and the other lights are unaffected.  

Percentage Closer Filtering: An optional 9x9 percentage closer filter replaces the single depth comparison with 81 comparisons around the lookup point, and the fraction of unoccluded samples gives the shadow factor and a soft penumbra. The depth bias is applied to every sample in the kernel. The texel step size is read from the shadow map's actual resolution, so the kernel stays correct when the resolution changes.  

Shadow Map Resolution: The resolution is selectable at runtime between 128 × 128 and 4096 × 4096 (128, 256, 512, 1024, 2048, and 4096). Changing it releases the old render texture, creates a new one, and rebinds it to the shadow camera, the shader, and the preview quad. At low resolutions the shadow edges become visibly blocky, and PCF can be used to compare how well it hides this at each size.  

Mirror Reflection: A mirror camera is positioned as the reflection of the main camera across the vertical mirror plane. Each frame, the main camera's view ray is intersected with the plane, the view direction is reflected about the plane normal, and the mirror camera is placed along the reflected ray and aimed at the intersection point. It renders into a render texture with the same size as the main camera's image. A replacement shader draws the scene for this camera in two steps. The mirror plane is drawn first, without writing depth, and sets a stencil value in the pixels it covers. All other objects are then drawn with the same full Blinn-Phong and shadow-mapping shaders as the main camera, but only where the stencil value is set. Reflected objects therefore keep their own material settings, respond to the lights, receive shadows, and honor the global rendering toggles. The mirror camera's position is passed to the lighting shader while it renders, so that specular highlights are computed from the reflected eye position, and the main camera's position is restored afterward.  

Reflection Compositing: A full-screen shader combines the two images. The mirror plane is drawn by the main camera with its own Blinn-Phong shading and an alpha of 0, while all other surfaces keep their normal alpha. The final color is `mainColor + mirrorColor * (1 - alpha)`, with the mirror image horizontally flipped to match the reflected camera. The mirror's own ambient, diffuse, and specular shading therefore stays visible on top of the reflected scene, and objects in front of the mirror hide the reflection. A mirror color setting controls the color shown in the reflection where no reflected geometry is present.  

Mirror Toggle: When the mirror is switched off, the mirror camera is disabled, and the plane is drawn with alpha 1 so that the compositing shader passes the main image through unchanged. The plane is then an ordinary Blinn-Phong surface that uses the same material parameters as when the mirror is on, so the plane's appearance stays consistent when switching. The mirror plane is excluded from the shadow map in both states, so the toggle does not change the shadows.  

Post-Processing Integration: The ground fog and torch effects from the previous assignment run on the main camera's image, using a world-position buffer rendered by a copy of the main camera, and can be combined in either order. Reflected geometry is rendered by the mirror camera and does not appear in that buffer, so the effects are not evaluated for it. Fog is therefore absent from the reflected scene, and the torch's blur and dimming can differ between an object and its reflection. These are limitations of applying screen-space effects after the fact.  

Per-Object Materials and Lights: Each object has its own Blinn-Phong material (ambient, diffuse, and specular coefficients and colors, and shininess) that can be edited at runtime. Up to five lights of point, directional, or spot type can be added, each with its own color, intensity, on/off state, and distance and angular attenuation. Point and spot lights are drawn as emissive spheres whose size and color follow the light's intensity and color.  

Runtime Control Panel: An on-screen panel built with Unity's immediate-mode GUI is generated automatically by reflecting over the public fields of the control scripts. Booleans become checkboxes, numeric fields become sliders that use their range limits, enums become selectors, and colors become sets of channel sliders. The panel has collapsible sections for the scene (camera, shadow map, mirror, and global render toggles), the lights (including position and rotation), the post-processing effects, and the per-object materials. It can be scaled, resized, dragged, and hidden with a key, and groups of debug options are shown as radio-style selections so only one is active at a time.  

# Screenshots:
<figure>
  <img width="449" height="251" alt="Show_Scene" src="https://github.com/user-attachments/assets/fb90c6e1-041a-48fb-a384-645e1c59c410" />
  <br />
  <figcaption>In this scene, we have introduced a vertical mirror and updated our shadows to use a shadow map rather than vector projections.</figcaption>
</figure>
<br />
<br />

<figure>
  <img width="600" height="335" alt="Shadow_Resolutions" src="https://github.com/user-attachments/assets/afaa6a3e-2346-4993-a26d-5374f821b35a" />
  <br />
  <figcaption>We can control the resolution of the shadow map that gets rendered to modify the quality of the shadows in scene.</figcaption>
</figure>
<br />
<br />

<figure>
  <img width="470" height="262" alt="PCF_Filter" src="https://github.com/user-attachments/assets/ff7ed637-0f89-49eb-833f-91bd53d0f152" />
  <br />
  <figcaption>We can use a 9x9 kernel-based Percentage Closer Filter (PCF) to soften and blur the edges of the shadows, removing pixelation.</figcaption>
</figure>
<br />
<br />

<figure>
  <img width="471" height="263" alt="Mirror_Controls" src="https://github.com/user-attachments/assets/c19d80c9-8d7e-41af-b01e-2745d1b6b006" />
  <br />
  <figcaption>We can control the color of the mirror and turn the effect off and on.</figcaption>
</figure>
<br />
<br />

# Code Release Restrictions:
Because this is a University of Washington Bothell course, I cannot publicly release my code solutions to these problems. However, code samples may be sent by request for hiring purposes (e.g. reviewing my previous work as it relates to a software engineering job).
