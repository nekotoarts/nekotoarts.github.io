---
name: Raytracing with Compute Shaders!
tools: [Godot, Compute Shaders, GLSL, HLSL]
image: /images/ComputeRaytracing/thumbnail_draft3.jpg
description: I was so excited to see compute shaders come to Godot 4! Shortly after the release of Godot 4 beta 1, I hopped right into experimenting with compute shaders and decided to try and create a simple raytracer!
---

# Raytracing with Compute Shaders!

I was really excited to hear the news for Godot 4! In July 2021, I was watching the [Godot Engine developer Q&A](https://youtu.be/g35xbKWF3ZQ?t=4739) and had asked in chat about planned features for shaders.

Specifically, I asked about plans for Geometry Shaders and Tessellation Shaders, to which [Hugo Locurcio (Calinou)](https://twitter.com/hugolocurcio?lang=en) responded about planned support for Compute Shaders in Godot 4!

![](/images/ComputeRaytracing/DevStream_CommentNekoto.png)
![](/images/ComputeRaytracing/DevStream_CommentCalinou.png)

I was extremely excited to finally be able to play around with compute shaders! Which is why when Godot 4 beta 1 finally released, I couldn't contain my excitement and immediately jumped into trying them out!

### Raytracing!

Through some exploring and trying to figure out how compute shaders were integrated into the engine, I eventually got a better understanding of the `RenderingDevice` and `RenderingServer` classes in Godot 4.

I prefer solidifying my learning experiences by trying to make a personal project using whatever I have learnt. So, after researching some of the uses for compute shaders for a while, I landed on the excellent raytracing series by Kuri (2018), which used compute shaders in Unity to create its raytracer!

{% include elements/figure.html image="/images/ComputeRaytracing/gpi-rt-teaser-sqr.png" caption="Raytraced Spheres by Kuri (2018)" %}

I thought this would be a fun project to try and recreate, and so I got to work on a raytracer!

Finally, I documented my experiences and shared what I had learnt by creating a tutorial for compute shaders in Godot 4. You can watch the video here:

{% include elements/video.html id="ueUMr92GQJc" %}

### Source Code

[![GitHub](https://img.shields.io/badge/github-%23121011.svg?style=for-the-badge&logo=github&logoColor=white)](https://github.com/nekotogd/Raytracing_Godot4)

## References and Resources Used

Kuri, D. (2018, May 3). _GPU Ray Tracing in Unity – Part 1_. Three Eyed Games. [https://www.gamedeveloper.com/programming/gpu-ray-tracing-in-unity-part-1](https://www.gamedeveloper.com/programming/gpu-ray-tracing-in-unity-part-1)

Möller, T., & Trumbore, B. (1997). _Fast, Minimum Storage Ray-Triangle Intersection_. Program of Computer Graphics. [https://fileadmin.cs.lth.se/cs/Personal/Tomas_Akenine-Moller/pubs/raytri_tam.pdf](https://fileadmin.cs.lth.se/cs/Personal/Tomas_Akenine-Moller/pubs/raytri_tam.pdf)

Scratchapixel. (2014, August 15). _Ray-Tracing a Polygon Mesh (Ray-Tracing a Polygon Mesh (Part 2))_. [https://www.scratchapixel.com/lessons/3d-basic-rendering/ray-tracing-polygon-mesh/ray-tracing-polygon-mesh-part-2](https://www.scratchapixel.com/lessons/3d-basic-rendering/ray-tracing-polygon-mesh/ray-tracing-polygon-mesh-part-2)

Whitaker, R. B. (2009, January 21). _Creating a Specular Lighting Shader_. RB Whitaker’s Wiki. [http://rbwhitaker.wikidot.com/specular-lighting-shader](http://rbwhitaker.wikidot.com/specular-lighting-shader)
