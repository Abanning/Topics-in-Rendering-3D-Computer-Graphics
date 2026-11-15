### MP3: Exploring Coordinate Spaces with Vanishing Effects
# Overview:
This project examines the stages of the graphics transformation pipeline by implementing the same "vanishing" effect in four different coordinate spaces: Object (OC), World (WC), Eye (EC), and Perspective (PC). In each space, every vertex of an object is linearly interpolated toward a target point using `p.xyz += w * (target - p.xyz)`, so that a weight of 0 leaves the object unchanged and a weight of 1 collapses it onto the target. A five-pass shader renders one result per space, plus the original geometry, and a runtime UI controls each pass independently. Key systems include:  

Five-Pass Shader: The shader contains one pass for each coordinate space (OC, WC, EC, PC) and a final pass that renders the original object. Each vanishing pass applies the interpolation at a different point in the transform chain and then completes the remaining transforms with explicit `mul()` calls. The OC pass interpolates object-space vertices before the object-to-world transform, the WC pass interpolates after it, the EC pass interpolates after the view transform, and the PC pass interpolates after the projection transform. Each pass tints the texture with a distinct constant color (red for OC, blue for WC, cyan for EC, and brown for PC) so that the result of every space can be told apart when several are shown at once. The original pass uses a user-selected color.  

Per-Pass Controls: Every pass has a weight slider and a Show toggle, which discards the pass's fragments when it is off. Each pass also has an Animate toggle, which ignores the weight slider and drives the weight from a Unity built-in time variable: `0.5 * (1 + _SinTime.z)` for OC, `pow(abs(_SinTime.z), WC_Rate)` for WC, `0.5 * (1 + _CosTime.z)` for EC, and `0.5 * (1 + _SinTime.y)` for PC. The WC_Rate exponent changes the shape of the oscillation.  

Vanishing Targets: By default, the OC pass vanishes toward the object-space origin, and a user-specified OC point can be enabled instead. The WC pass can vanish toward the position of any scene GameObject, so the target can be moved during play, or toward the OC point transformed into world space. The EC pass can use the OC point or the WC point transformed into eye space. The PC pass uses a user-defined point in perspective space, or the OC or WC point transformed through the full pipeline into perspective space. Because all targets can be expressed in every space, the same vanishing result can be reproduced in different spaces. Every vertex position and target involved in a computation must be in the same space.  

Eye Space Z-Only Mode: The EC pass includes an option that applies the interpolation only to the z component of each vertex. Because this moves the object along the camera's viewing direction, an object can be pushed from behind an occluder to in front of it, and its projected screen position shifts accordingly.  

Homogeneous Coordinates in Perspective Space: Because the PC pass operates after projection, each vertex is divided by its homogeneous w component to obtain normalized device coordinates, interpolated toward the target, and then multiplied by w again so that the hardware receives a valid clip-space position.  

Bit-Mask Flag Encoding: The Show, Animate, and target-selection options for all passes are packed into a single 17-bit integer bit-mask. A script (FunctionEncode) encodes the mask each frame and sends it to the shader along with the weights, rates, target points, and original-pass color as global shader properties. The shader decodes the mask using bitwise AND operations to select its behavior in each pass.  

# Screenshots:
<figure>
  <img width="600" height="337" alt="Color" src="https://github.com/user-attachments/assets/e0227d3a-31de-4111-bb28-95538b28d520" />
  <br />
  <figcaption>Color controls on the teapot texture.</figcaption>
</figure>
<br />
<br />

<figure>
  <img width="600" height="337" alt="OC" src="https://github.com/user-attachments/assets/db32da8c-4dce-4735-aa95-6382bc23cdb5" />
  <br />
  <figcaption>Object coordinate space - Vanishing into itself @ (0,0,0).</figcaption>
</figure>
<br />
<br />

<figure>
  <img width="600" height="337" alt="WC" src="https://github.com/user-attachments/assets/bca50259-e220-43d8-b278-1465db7f51e3" />
  <br />
  <figcaption>World coordinate space - Vanishing into an object in the world.</figcaption>
</figure>
<br />
<br />

<figure>
  <img width="600" height="337" alt="EC_withOverride" src="https://github.com/user-attachments/assets/91726d38-8003-4a64-9f18-16171b1e384e" />
  <br />
  <figcaption>Eye coordinate space - Vanishing beyond the camera frustum's far view plane. Also demos overriding EC coordinate with object coordinate (OC) @ (0,0,0).</figcaption>
</figure>
<br />
<br />

<figure>
  <img width="600" height="337" alt="PC" src="https://github.com/user-attachments/assets/6879a294-4f72-41dc-bdc1-3bf79ff2f808" />
  <br />
  <figcaption>Perspective coordinate space - Vanishing in perspective space (notice the slower movement compared to the others). Manual override with a slider to control X and Y values of clip position.</figcaption>
</figure>
<br />
<br />

# Code Release Restrictions:
Because this is a University of Washington Bothell course, I cannot publicly release my code solutions to these problems. However, code samples may be sent by request for hiring purposes (e.g. reviewing my previous work as it relates to a software engineering job).
