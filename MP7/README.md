### MP7: Post-Processing with Ground Fog and Torch Volumetrics
# Overview:
This project extends the Blinn-Phong and projected-shadow renderer from the previous assignments with world-space post-processing. After the scene is rendered, full-screen shaders use a per-pixel world-position buffer to add two effects: a height-based ground fog that fills the entire ground plane, and a "torch" effect that keeps a region of the scene in focus while blurring and dimming the rest, with a volumetric glow around the focus region. The effects can be shown individually or chained in either order, and all of their parameters can be adjusted at runtime. Key systems include:  

World-Position Buffer: A second camera copies the main camera's position, rotation, field of view, clip planes, and projection matrix every frame and renders the scene into a floating-point render texture using a replacement shader. For each pixel, the shader stores the world-space position of the visible surface in the RGB channels and its distance from the camera in the alpha channel. The post-processing shaders read this texture to recover the 3D position of every pixel without re-rendering the scene. Pixels with a distance of zero are treated as background and are left unchanged by the fog.  

Post-Processing Pipeline: A post-processing script runs when the main camera finishes rendering and passes the rendered image through the selected full-screen materials. Five modes are supported: no effect, ground fog only, torch only, ground fog followed by torch, and torch followed by ground fog. The two combined modes use an intermediate render texture to chain the passes. Because the second effect operates on the output of the first, the two orders produce different results.  

Fog Model: Both effects use the same participating-media model. The transmittance is `T = e^(-τ)`, where `τ = σD` is the extinction coefficient `σ` times the normalized length `D` of the viewing ray that travels through the medium. A dropoff parameter reshapes the optical depth with `τ' = smoothstep(0, 1, pow(τ, dropoff))`. The color added by the medium is `FogColor * Scattering * (1 - T)`, and the final pixel color is `T * ObjectColor + computedFogColor`, so the fog attenuates the color of the object behind it and adds its own in-scattered color.  

Ground Fog: The ground fog is computed in a single full-screen pass without any fog geometry. For each pixel, the shader builds the viewing ray from the camera position through the pixel's world position and intersects it with two horizontal planes: the top of the fog at the fog height, and the ground plane at y = 0. The visible interval of the ray inside the fog runs from the top plane (or the camera, if it is inside the fog) to either the ground plane or the visible surface, whichever comes first, so objects that stick up through the fog are only partially immersed. The interval is divided by the fog height to get `D`. The fog is applied to visible surfaces within user-defined X and Z bounds, and the effect is designed for a camera looking downward at the ground, so rays that point upward or horizontally are left unchanged.  

Animated Ground Fog: As an extra-credit feature, the fog height can be animated with a sum of two sine waves, one along X and one along Z, each with its own speed, amplitude, and frequency. The wave is evaluated at the world-space X and Z position of each visible point and varies with time, so the fog surface appears to roll across the ground.  

Torch Focus Region: The torch is a user-positioned volume with a radius and a far distance, and it can be either a sphere or an axis-aligned cube. For each pixel, the shader measures the distance from the visible world position to the torch. Inside the torch radius, the original color is kept. Beyond the far distance, the color is replaced with a 15x15 box-blurred version of the image, multiplied by a far-intensity factor so that it is also dimmed. Between the two distances, the original and blurred colors are blended with `smoothstep(0, 1, 1 - n² / f²)`, where `n` is the distance past the torch radius and `f` is the distance between the torch radius and the far distance. A Reverse Blur option swaps the behavior so the inside of the torch is blurred and dimmed instead.  

Torch Volumetric Glow: Around the torch, a volumetric effect is added using the same fog model, with its own extinction, dropoff, scattering, and color. The shader casts the viewing ray from the camera through each pixel and intersects it with the torch volume, using a ray-sphere quadratic solution or a ray-box slab test, and clips the interval at the visible surface. The interval is normalized by half the torch radius for the sphere, or by the cube's diagonal for the cube, to produce `D`. The blur and the volumetric glow can each be switched on or off independently.  

Debug Visualizations: Each shader can display its intermediate values as grayscale or color images. Both effects can show the optical depth `τ'`, the normalized interval `D`, the transmittance `T`, and the computed fog or torch color. The ground fog can also show a region map of the fog volume (above the fog height, outside the X or Z bounds, or inside the fog), and the torch can show a region map in which the inside of the torch is green, the transition region shows the blend factor, and the fully blurred region is red.  

Scene Control: A single SceneControl component gathers the parameters of the ground fog, the torch, the post-processing mode, the camera controls, the shadow toggles, the global rendering switches, and the lights, and pushes them to the corresponding control scripts every frame. These scripts then upload the values to the materials. The Blinn-Phong lighting and projected shadows from the previous assignment remain in place, and a shadow manager combines each light's On/Off state with a user toggle to decide which shadows are drawn.  

# Screenshots:
<figure>
  <img width="443" height="247" alt="Spot_Light_Controls" src="https://github.com/user-attachments/assets/8eb37a41-6030-4d84-9ba9-5c4e4ae770d5" />
  <br />
  <figcaption>Spot light angular attentuation and distance attenuation are fully configured. The edges of the light respond based on a dropoff equation and the cone angles provided. Shadows move when the light moves inreal time.</figcaption>
</figure>
<br />
<br />

<figure>
  <img width="600" height="335" alt="Ground_Fog" src="https://github.com/user-attachments/assets/a9c5c0e1-2560-4e44-9f24-0280f33e2097" />
  <br />
  <figcaption>This project implements post processing ground fog which responds to the height, dropoff, extinction, and scattering sliders to look realistic with reflecting light and scattered light.</figcaption>
</figure>
<br />
<br />

<figure>
  <img width="600" height="335" alt="Ground_Fog_Animation" src="https://github.com/user-attachments/assets/2c5d8cd4-3958-4fba-ad28-cd06b8d4419e" />
  <br />
  <figcaption>The ground fog also supports animation, using Sin(time) to dynamically move the surface height with respect to x and z axes. Adjusting the freq and amiplitude in the different regions can have cool effects on what the fog looks like in the scene.</figcaption>
</figure>
<br />
<br />

<figure>
  <img width="600" height="335" alt="Post_Processing_Modes" src="https://github.com/user-attachments/assets/0bde858d-985a-4ba9-a538-2d22c3fc9043" />
  <br />
  <figcaption>Demonstrating how to select different post processing modes.</figcaption>
</figure>
<br />
<br />

<figure>
  <img width="600" height="335" alt="Basic_Torch_Controls" src="https://github.com/user-attachments/assets/d71170be-2e25-4854-9c0e-a198ccfc7930" />
  <br />
  <figcaption>This project also supports a Torch shader, or spherical volumetric fog that illuminates within and blurs and dims outside.</figcaption>
</figure>
<br />
<br />

<figure>
  <img width="600" height="335" alt="Torch_Fog_Controls" src="https://github.com/user-attachments/assets/59e48c1c-0c25-4278-be6b-d65c7ef12867" />
  <br />
  <figcaption>Demonstrating the torch fog controls, which also include dropoff, extcinction, and scattering. Can control the blur effect inside and outside of torch and the distance cutoff for the blurring and dimming effects.</figcaption>
</figure>
<br />
<br />

<figure>
  <img width="600" height="335" alt="Torch_Cube" src="https://github.com/user-attachments/assets/37780193-79f1-4b56-9c30-2b16558b06cd" />
  <br />
  <figcaption>Extra Credit - Included a cubic torch volume, which involved more complex intersection math and interesting edge cases with how the fog behaves looking through a cube instead of sphere.</figcaption>
</figure>
<br />
<br />

# Code Release Restrictions:
Because this is a University of Washington Bothell course, I cannot publicly release my code solutions to these problems. However, code samples may be sent by request for hiring purposes (e.g. reviewing my previous work as it relates to a software engineering job).
