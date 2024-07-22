---
title: Read Depth Texture (Godot 3)
date: 2022-08-23
description: "How to read the depth texture in Godot 3"
tags: [Shaders, Godot]
style: border
color: danger
---

```glsl
float depth = texture(DEPTH_TEXTURE, SCREEN_UV).r;
vec3 screen_coords = vec3(SCREEN_UV, depth);
vec3 ndc = screen_coords * 2.0 - 1.0;
vec4 view_space = INV_PROJECTION_MATRIX * vec4(ndc, 1.0);
view_space.xyz /= view.w;
depth = -view.z;
```
