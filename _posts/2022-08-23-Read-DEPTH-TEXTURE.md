---
title: "How do I lookup the depth texture?"
description: "The depth buffer's values seem really weird when you sample it like a regular texture. How do you even get the linear depth value that you're looking for?"
date: 2022-08-23
tags: [Shaders, Godot, Rendering]
style: border
color: danger
---

![](/images/DepthTextureArticle/depth_buffer.png)

Here's a short tutorial on reading the `DEPTH_TEXTURE` in Godot! I remember this being one of the most confusing things to understand when beginning to write shaders, so, I decided to make a small written breakdown here.

The depth buffer contains a bunch of floating-point values that hold the depth of every single pixel in the scene. These values are stored on the GPU's memory in a texture, called the 'depth texture'.

Godot already provides this to us natively in all **spatial** shaders. Which means we cannot sample the depth texture in `canvas_item` or `particles` shader types.

## Sampling the Depth Texture

The depth texture holds the (transformed) depth values for the scene. These are all floating-point values, so only a single channel of the depth texture is actually filled with data. This channel is the first channel (the red channel) of the depth texture.

In the `fragment()` function, we sample it as follows:

```glsl
float depth = texture(DEPTH_TEXTURE, SCREEN_UV).r;
```

Notice the `.r` that allows us to take just the red channel of the depth texture. Also, we use `SCREEN_UV` because this has the same UVs as the `SCREEN_TEXTURE`.

This depth value is totally unusable for most situations however, because this isn't the linear depth of the scene. Instead, its currently on a logarithmic scale, and we need to transform first.

![](/images/DepthTextureArticle/raw_depth_texture.png)
_Raw depth texture value when we read it_

![](/images/DepthTextureArticle/depth_raised_to_50.png)
_Raising the power of this depth to 50 shows some details_ `pow(depth, 50.0)`

This occurs because the vertices of the mesh are transformed into clip-space using the `PROJECTION_MATRIX` which makes the "z" value non-linear. Of course, to counteract this, we need to just multiply it by the `INV_PROJECTION_MATRIX` so we can head back into view-space.

If you're new to shaders, take a look at the different vertex transformation stages below:

![](/images/DepthTextureArticle/coordinate_systems_dark.png)
_Figure 1. Vertex Transformation Stages (Vries, 2014)._

We currently have a clip-space value for the depth texture, so we want to get back to view space by undoing the projection transformation.

Right now, the `depth` value that we have is between `0` and `1`, but completely non-linear. Also, don't forget that we're working in clip-space, so we have to construct NDC (normalized device coordinates) that runs from `-1` to `1`:

> Note for Godot 4 users: Clip-space normalized device coordinates (NDC) used to run from $[-1,1]$ on all axes in OpenGL (Vries, 2014), however, Vulkan handles NDCs a little differently.
>
> The $x$ and $y$ axes still run from $[-1, 1]$ but the $z$ axis runs from $[0,1]$ (Wellings, 2016).

```glsl
vec3 screen_coords = vec3(SCREEN_UV, depth) // Still between 0 and 1;
vec3 ndc = screen_coords * 2.0 - 1.0 // Now between -1 and 1;
```

The `vec3(SCREEN_UV, depth)` is some GLSL syntax sugar that is equivalent to `vec3(SCREEN_UV.x, SCREEN_UV.y, depth)`

Now, let's make this linear again by transforming them:

```glsl
vec4 view_space = INV_PROJECTION_MATRIX * vec4(ndc, 1.0);
view_space.xyz /= view.w;
depth = -view.z;
```

Behold! You now have the linear value of scene's depth! Here's the full code if you want to come back later and just copy it:

```glsl
float depth = texture(DEPTH_TEXTURE, SCREEN_UV).r;
vec3 screen_coords = vec3(SCREEN_UV, depth);
vec3 ndc = screen_coords * 2.0 - 1.0;
vec4 view_space = INV_PROJECTION_MATRIX * vec4(ndc, 1.0);
view_space.xyz /= view.w;
depth = -view.z;
```

## References

Wellings, M. (2016, March 20). The New Vulkan Coordinate System. Retrieved September 10, 2022, from [https://matthewwellings.com/blog/the-new-vulkan-coordinate-system/](https://matthewwellings.com/blog/the-new-vulkan-coordinate-system/)

Vries, J. D. (2014, June). _Coordinate Systems_. LearnOpenGL. Retrieved August 23, 2022, from [https://learnopengl.com/Getting-started/Coordinate-Systems](learnopengl.com/Getting-started/Coordinate-Systems)
