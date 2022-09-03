---
title: "Fluffy 2D Pixel Art Trees!"
collection: teaching
excerpt: "Learn how to make Fluffy looking 2D Pixel Art Trees in Godot. I'll walk you through all the shader code! <br/><img src='https://cdn.hashnode.com/res/hashnode/image/upload/v1627978882209/mgLz3aIaU.png'>"
type: "Shader Tutorial"
permalink: /teaching/fluffy-2d-pixel-art-trees
date: 2021-08-03
---

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1627978882209/mgLz3aIaU.png)

Let's make some fluffy looking 2D pixel art trees in Godot!

## Assets I will be using

The Tree from this free asset pack on [itch.io](https://itch.io)

[Pixel Art Top-Down Basic by Cainos](https://cainos.itch.io/pixel-art-top-down-basic)

## My Pixel Art is already good, do I need to do this?

Not necessarily, but it will put a lot of life into your tree when placed in the game world.

For most RPGs, trees make up a lot of their map since they're such basic vegetation for the world. Due to this, player see trees quite often.

Static Trees may look good but are no fun, all you can really do is bump into them but they don't feel alive at all!

Real Trees have leaves that wave around as the wind blows and also shed leaves sometimes.

## How do we juice our Trees?

We're going to make 2 main changes:

-   Waving Leaves
-   Flying Leaves

Very simple, very juicy, and very much alive!

### One small initial step

We need to head back to our pixel art program, I will be using Aseprite but you can use whatever you like.

Open up your Tree and separate the Leaves and the Bark into separate layers like so:

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1627973859014/Y4clxM2k2.png)
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1627973872632/g_7lPl_RX.png)

Now export them as separate images: `bark.png` and `leaves.png`

The names of the file are not important, just that they are separate files

## Waving Leaves

In Godot, create a `StaticBody2D` for our Tree, and 2 sprites - the Bark and the Leaves

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1627974388277/_CK_tmtGS.png)

Position that Bark, such that its origin point is at the bottom of the Bark:

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1627974458740/hszdY2532.png)

Now bring in the Leaves and align it with the tree

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1627974495066/dbZBBQAnX.png)

On the Leaves Sprite, go to the `Materials` section in the Inspector and create a new `ShaderMaterial`

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1627974576642/xga7DlCbW.png)

Open up the `ShaderMaterial` and create a new `Shader`

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1627974623810/FgDiveFar.png)

Now we're ready to make our leaves wave! Let's start by defining our shader to be a 2D shader

```glsl
shader_type canvas_item;
```

To make our leaves wave, we need to make sure that each pixel of the shader has a pseudo-random distortion, so you know what that means - its time for some noise textures!

Define 3 textures:

-   Our leaves albedo texture
-   2 noise textures

```glsl
uniform sampler2D albedo_texture : hint_albedo;
uniform sampler2D noise_texture : hint_albedo;
uniform sampler2D noise_texture2 : hint_albedo;
```

We're using 2 noise textures for increased variation in our waving effect.

Now for the parameters that control our waves:

```glsl
uniform float distortion = 0.058;
uniform vec2 offset = vec2(0.0);
uniform float time_scale = 0.2;
```

`distortion` will control the strength of the waves
`offset` is used to re-align the texture after the UV offset has been added by the distortion
`time_scale` controls how fast the waves are

Now lets make all of it actually work.

Create the `fragment` function:

```glsl
void fragment(){

}
```

Inside the fragment function, lets get the color of our 2 noise textures. This can be done using the `texture` function which performs a texture lookup at the specified UV coordinate. The current UV is already given to us as a variable we can use, so we simply write:

```glsl
vec3 noise_color = texture(noise_texture, UV).rgb * distortion;
vec3 noise_color2 = texture(noise_texture2, UV).rgb * distortion;
```

We multiply them with `distortion` in order to control their strength.

However, we want the noise to move, according to our `time_scale` uniform. To do this, we use the built-in `TIME` variable and multiply it with our `time_scale` uniform, then we add this to our UV.

But then both of our noise textures would move in the same way, and there would be effectively no variation between the textures. To combat this, we will move the second noise texture only vertically.

```glsl
vec3 noise_color = texture(noise_texture, UV + TIME * time_scale).rgb * distortion;
vec3 noise_color2 = texture(noise_texture2, vec2(UV.x, UV.y - TIME * time_scale)).rgb * distortion;
```

Now let's take the average of both these noise textures when overlayed on top of each other:

```glsl
vec3 combined_noise = noise_color + noise_color2;
combined_noise /= 2.0;
```

Finally, we use our combined noise to distort the UVs of the leaves texture, but make sure to add our offset as without it, our texture will be moved into an odd position and will be clipped by the sprite node.

```glsl
COLOR = texture(albedo_texture, UV + combined_noise.rr + offset);
```

Here's the whole code if you want to copy it over:

```glsl
shader_type canvas_item;

uniform sampler2D albedo_texture : hint_albedo;
uniform sampler2D noise_texture : hint_albedo;
uniform sampler2D noise_texture2 : hint_albedo;

uniform float distortion = 0.058;
uniform vec2 offset = vec2(0.0);
uniform float time_scale = 0.2;

void fragment(){
	vec3 noise_color = texture(noise_texture, UV + TIME * time_scale).rgb * distortion;
	vec3 noise_color2 = texture(noise_texture2, vec2(UV.x, UV.y - TIME * time_scale)).rgb * distortion;
	vec3 combined_noise = noise_color + noise_color2;
	combined_noise /= 2.0;
	COLOR = texture(albedo_texture, UV + combined_noise.rr + offset);
}
```

### Setting up the material

We need to actually put noise textures for our material, as well as the leaves texture.

In the Inspector, you should see a new section called `Shader Param`

Open it up and place the Leaves texture in the `Albedo Texture` slot.

As for the noise textures - click on them and select `New NoiseTexture`

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1627976063210/IH3o5ydos.png)

Open it up, and check the `Seamless` option
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1627976095305/qZAERt3LnW.png)

For the noise, click and select `New OpenSimplexNoise`
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1627976134587/zPP5XecST.png)

Open up the noise and raise the `Octaves` to 6

Repeat all this for the other noise texture but **set the seed to a different number on the second noise texture**

What it should look like now:

<iframe width="560" height="315" src="https://www.youtube.com/embed/pmMnoRTdGCI" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Flying Leaves

Open up your Pixel Art program and make a very small leaf sprite. Make sure that the leaf's color is plain white `#FFFFFF`

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1627976339251/j_6FIgU7a.png)

This is important because we are going to set the color in Godot and not here.

Back in Godot, create a new `Particles2D` node and make sure it is emitting. Set the amount to a low number like 4.

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1627976474688/uawhAsuB5.png)

In the Inspector, under the `Process Material` section, create a new `ParticlesMaterial` and open it up, do this the same way we create our `ShaderMaterial` from earlier.

You should now see a bunch of white dots falling. However, we don't want them to fall.

Open up the `ParticlesMaterial` and go to the `Gravity` section. Set the gravity to 0 for all the axis.

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1627976633021/0SlTEwl7hs.png)

Open up the `Direction` section and set the spread to a low angle, such as 20 so that out leaves don't look erratic.

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1627976676648/5hXPEKI81.png)

Now lets make them move. Head over to the `Initial Velocity` section and give them some initial velocity

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1627976780223/R4RUOjtEg.png)

Also lets make them rotate at random speeds. Go to the `Angular Velocity` section and set the velocity to max, as well as the randomness to max.

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1627976846737/D-wEINMJs.png)

They should also spawn at random angles, so go to the `Angle` section and do the same thing.

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1627976899466/p5yqGdo8-.png)

Now for the last 2 steps. Open up the `Textures` section in the Inspector (Not in the `ParticlesMaterial`) and place your leaf texture in there.

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1627977026260/cS0NxocR7.png)

Head back to the `ParticlesMaterial` and adjust the `Scale` parameter so that it fits your tree.

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1627977095478/FbRIV7Y1S.png)

The leaves just disappear after a certain distance which looks a little weird. Let's fix that. You may have noticed the `Scale Curve` property under the `Scale` section. Open it up and create a new `CurveTexture`.

If there is no `Curve` already created, make one and open it up. You should see this now:

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1627977276552/5_mvbFiAc.png)

Click on the second point, and drag it down to the bottom
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1627977330303/nWtwG3LhY.png)

Now adjust the handle so the the curve takes longer to drop:
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1627977393602/rqHFQxy1v.png)

Now our leaves shrink out which looks much better!

Finally, let's fix the color of our leaves. Head over to the `Color` section and open up the color editor.

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1627977467529/IEuPNv-Jv.png)

You should see a little color-picker icon, use it to select the color of your tree.

There we go! A much more lively looking pixel art tree!

Final Product

<iframe width="560" height="315" src="https://www.youtube.com/embed/MSqttc_IHyE" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## My Links

Find my games on [itch.io](https://nekotoarts.itch.io)

Find my work on [GitHub](https://github.com/nekotogd)
