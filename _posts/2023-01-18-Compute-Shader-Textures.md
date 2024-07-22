---
title: Everything About Textures in Compute Shaders!
description: Not enough documentation on using textures in Godot 4 compute shaders? Here's everything you need to know
tags: [Shaders, GLSL, Compute Shaders, Godot]
date: 2023-01-18
style: border
color: warning
---

![](/images/ComputeShaderTextures/thumbnail_draft1.png)

Godot 4 is now racing towards its stable release, and I've been playing around with the new compute shader system it includes now. However, there isn't quite enough documentation surrounding them yet, and most sources are only covering how to use storage buffers.

I made a video covering an [Introduction to Compute Shaders](/portfolio/compute-shader-raytracer) but avoided talking about textures because I was still getting used to using them, and beta 1 had even less information surrounding them than we have right now.

So here's everything that you'll need to know about using textures in a compute shader for Godot 4, organized by usage scenario.

## Writing to a Texture

Suppose you've got data in a compute shader that you want to store as colors in a texture. First, we'll declare a writeable texture similar to how we declare storage buffers.

```glsl
layout(set = 0, binding = 0, rgba32f) uniform image2D OUTPUT_TEXTURE;
```

The cool thing about compute shaders is that we can output to multiple textures at once, so feel free to add as many as you need.

To write out color data from the compute shader we use an `image2D`. Notice in the declaration that we also specify the image format of `rgba32f`, which means that our image data is stored as 4 channels per-pixel with each channel consisting of a 32-bit floating-point value. This format corresponds to us setting pixel data using a `vec4`.

You can see more about all the different possible formats [here](<https://www.khronos.org/opengl/wiki/Layout_Qualifier_(GLSL)>).

Now, somewhere within your compute shader you have successfully calculated the color you want to store in the image. Before setting it however, we need to define what position in the image we want to write to.

In fragment shaders, we're used to giving output colors using UV coordinates, but in compute shaders, we need to define the integer-coordinate of our texel (pixel on a texture = texel). This is because the image doesn't come with UV data, we only know that it has a grid of texels, and we can write to any of them.

Suppose your compute shader is running 1 invocation for every pixel, then we know that `gl_GlobalInvocationID.xy` corresponds exactly to the texel we want to write to. Setting the color is pretty simple then:

```glsl
vec4 color; // Calculated by Compute Shader
ivec2 texel = ivec2(gl_GlobalInvocationID.xy);
imageStore(OUTPUT_TEXTURE, texel, color);
```

Depending on where in the image you want to write to, and how you're compute shader is handling its calculations, you'll have to figure out where the texel you want to write to is located.

### Attaching the Texture in GDScript

For declaring any texture to attach to a compute shader, you'll need:

-   A texture format (`RDTextureFormat`)
-   A texture view (`RDTextureView`)
-   Image Byte Data

Even if the texture is an output, it will need to be initialized with some byte data. We'll start by creating a texture format and assigning the values we need:

```gdscript
var fmt := RDTextureFormat.new()
fmt.width = output_texture_width
fmt.height = output_texture_height
fmt.format = RenderingDevice.DATA_FORMAT_R32G32B32A32_SFLOAT
fmt.usage_bits = RenderingDevice.TEXTURE_USAGE_CAN_UPDATE_BIT | RenderingDevice.TEXTURE_USAGE_STORAGE_BIT | RenderingDevice.TEXTURE_USAGE_CAN_COPY_FROM_BIT
```

The format parameter must **exactly** match what we set in our shader. Since we used `rgba32f` in the shader, we do the same for the texture format: `RenderingDevice.DATA_FORMAT_R32G32B32A32_SFLOAT`.

You're probably wondering what those usage bits are right? They tell Vulkan what kind of operations to expect will be done on this texture. Specifically, `RenderingDevice.TEXTURE_USAGE_CAN_COPY_FROM_BIT` is extremely important because it allows the CPU to read back whatever the output of the compute shader is!

Next up we'll create the texture view. It only contains settings regarding swizzling and overriding the format, so there's nothing for us to set, but we still need it anyways.

```gdscript
var view := RDTextureView.new()
```

The last thing we need to prepare is the initialization byte data. We'll do that by creating a new, empty `Image` and then pulling the byte data from there. Finally, we'll ask our `RenderingDevice` to create a texture for us (`rd` is a `RenderingDevice` instance).

```gdscript
var output_image := Image.create(output_texture_width, output_texture_height, false, Image.FORMAT_RGBAF)
var output_tex = rd.texture_create(fmt, view, [output_image.get_data()])
# You should store "output_tex" as a global variable so you can retrieve its output data later
var output_tex_uniform := RDUniform.new()
output_tex_uniform.uniform_type = RenderingDevice.UNIFORM_TYPE_IMAGE
output_tex_uniform.binding = use_same_binding_from_your_shader_code
output_tex_uniform.add_id(output_tex)
```

Again, note that the `Image` we created needs to have the same format `Image.FORMAT_RGBAF`. Additionally, since we're using an `image2D` in the shader, our uniform type is set to `RenderingDevice.UNIFORM_TYPE_IMAGE`.

Now you can add this `output_tex_uniform` to the binding list in the same way you add storage buffers to the binding list.

## Sampling a Texture in a Compute Shader

Define the input texture similar to how we declare storage buffers:

```glsl
layout(set = 0, binding = 0) uniform sampler2D INPUT_TEXTURE;
```

Then just sample the texture the same way you would as in a fragment shader. You can calculate UV coordinates by doing `uv = texel_position / texture_resolution`. Also you can find the resolution of a `sampler2D` using `textureSize(TEXTURE)`, or find the resolution of an `image2D` using `imageSize(IMAGE)`.

```glsl
vec4 color = texture(INPUT_TEXTURE, UV);
```

### Attaching the Texture and Sampler in GDScript

Sampler types are a little different from just attaching textures to compute shaders. We have to create a uniform that holds two IDs:

-   The type of texture sampler to be used by the shader
-   The texture to be sampled

Creating the sampler is done using the `RDSamplerState` class:

```gdscript
var sampler_state := RDSamplerState.new()
var sampler = rd.sampler_create(sampler_state)
```

Next, let's load up the texture we want to sample in the shader:

```gdscript
var image_file : Texture2D = load("res://path/to/image.png")
var image := image_file.get_image()
image.convert(Image.FORMAT_RGBAF)
```

Our `sampler2D` type reads back a `vec4`, which is 4-channel 32-bit floating-point value. That corresponds to the `Image.FORMAT_RGBAF`, which is why we're converting the image before using it.

Now let's create the texture format and texture view, mostly the same as we did before:

```gdscript
var fmt = RDTextureFormat.new()
fmt.width = image.get_width()
fmt.height = image.get_height()
fmt.format = RenderingDevice.DATA_FORMAT_R32G32B32A32_SFLOAT
fmt.usage_bits = RenderingDevice.TEXTURE_USAGE_CAN_COPY_FROM_BIT | RenderingDevice.TEXTURE_USAGE_SAMPLING_BIT | RenderingDevice.TEXTURE_USAGE_CAN_UPDATE_BIT
var view = RDTextureView.new()
```

Notice that this time we also added the `RenderingDevice.TEXTURE_USAGE_SAMPLING_BIT`.

Finally, let's create the texture and add both the sampler and texture to a uniform so we can pass that into the binding list:

```gdscript
var tex = rd.texture_create(fmt, view, [image.get_data()])
var sampler_uniform := RDUniform.new()
sampler_uniform.uniform_type = RenderingDevice.UNIFORM_TYPE_SAMPLER_WITH_TEXTURE
sampler_uniform.binding = 0
sampler_uniform.add_id(sampler)
sampler_uniform.add_id(tex)
```

We choose the uniform type `RenderingDevice.UNIFORM_TYPE_SAMPLER_WITH_TEXTURE` as that's what we defined in our shader. Now you can add this `sampler_uniform` to the binding list in the same way you add storage buffers to the binding list.

## Texture Readback and Texture Updating

Suppose you have two textures passed to your compute shader. One that it writes to, and one that it reads from. I'll show you how to copy the data from one texture into another, which will teach you how to read and write to a texture from the CPU side.

Let's call the texture being written to by the compute shader as the `output_tex`, and the one that it is sampling will be called `sampler_tex`.

Now, `output_tex` contains all the output data from the last time we dispatched the compute shader, and we want to put all that data into `sampler_tex` so that it can use it on the next dispatch.

First, let's readback whatever was written to `output_tex` by our compute shader:

```gdscript
var byte_data : PackedByteArray = rd.texture_get_data(output_tex, 0)
```

Pretty simple right? The `0` over there means that we are reading from texture layer `0`. If you have multiple layers, you pass in whatever layer you wish to read from. If you want to display what's in here, you can use the `Image` class:

```gdscript
var byte_data : PackedByteArray = rd.texture_get_data(output_tex, 0)
var image := Image.create_from_data(output_tex_width, output_tex_height, false, Image.FORMAT_RGBAF, byte_data)
```

Then attach that image to a texture or save it to the disk to view it.

Writing data is pretty easy too! If we have byte data that we want to upload to one of the textures, all we need to do is:

```gdscript
rd.texture_update(sampler_tex, 0, byte_data)
```

So copying data from one texture to another is as simple as one line of code:

```gdscript
rd.texture_update(old_frame_tex, 0, rd.texture_get_data(output_tex, 0))
```

Keep in mind that this **only works if both textures have the same dimensions**. You cannot copy data from one texture into another unless they have the same dimensions. Uploaded data must always match the resolution that was declared in the texture format when creating the texture object.

Now you know how to create textures, write to them in a compute shader, sample them in a compute shader, read back the data in GDScript, and also update or copy them in GDScript.

I hope this helped anyone who's trying to fiddle around with compute shaders!
