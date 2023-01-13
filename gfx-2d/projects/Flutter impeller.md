#TODO 

Source: https://github.com/flutter/engine/tree/main/impeller

- looks rather simple overall
- There is a Scene representation (`impeller/scene/scene.h`)
- Uses a canvas-like API internally (`impeller/aiks/canvas.h`)
- Uses tessellated mesh (libtess2)
- it has its own high level gpu api abstraction layer on top of vulkan, dx, metal and gles
- a GLSL to backend specific shaders translator
- dedicated rounded rect shadow (`rrect_shadow_contents`, `rrect_blur.frag`)
- support for sdf and non-sdf glyphs
- support for skinned meshes (`impeller/scene/skin.h`)
- support for PBR materials and lighting (`impeller/scene/material.h`)
- they use the stencil buffer for some stuff
	- clipping
	- ??
- MSAA for some things
	- I guess path AA in general since they use libtess2?
- shaders are in `impeller/entity/shaders`

## Terminology

- **contents**: virtual class for a high level drawing operation (for example a filled path, or a rounded rect shadow or a clip), wrapped into an entity.
- **entity**: a high level drawing operation, wraps a contents, stores bit of common state for like transforms and blend mode to avoid putting it in contents I suppose.