This page explains how to extract maps from Destiny 2 into Unreal Engine 5. I begin with the actual method of Charm -> UE5, then later discuss some of how it works and improvements for the future.

## How to import maps into UE5

First open the Configuration panel and set your preferred ue5 import path. It is recommended to not use the top-level `Content/` directory and instead a child of this, such as `...Content/CharmMaps/`. I recommend making sure that "Use single folder extraction for maps" is `True` as textures are highly reused between similar regions, and so you don't have 10gb of the same textures saved.

Then you can export the map you want by going to Activities. If you use "Select all", it will extract every map shown in the list. Each map will export the `.fbx`, an info `.cfg` file, and a `.py` file.

You'll need UE5 installed, as well as the `Python Editor Script Plugin`. Once this is installed, navigate to the output log and select `Python` where it says `Cmd`. In the box to the right, you then enter the path of the `.py` file eg: `C:\Charm\export\Maps\4834C680_import_to_ue5.py`. The map export should then show some things on the screen. After a while (and large maps can take a long time on slow PCs) it should import all statics and materials, assign the materials to the statics, and assemble the map into a new level.

If you have any problems, I recommend the [Destiny Model Rips (DMR)](https://discord.gg/RRbYsaC4h6) discord server. If you encounter Charm or Python errors, feel free to begin an Issue while copying the `charm.log` and the Python error encountered.

If you want to only do a specific part of the importing process (like only assign materials and skip assembling and importing the model) you can do so by finding the relevant function in the script and commenting out the parts you don't want to run. A good example of this is only wanting to see the map assembled, but without any materials or shaders (as these take the longest to load).

If you want to recompile all the materials and replace the shader code inside each material, uncomment the last line that says `importer.update_material_code()`.

## Importing other things like entities or individual statics

It works practically the same, I automatically change the .py file to mirror the choices made in Charm. No extra work should be required.

## How this works

Destiny 2 on PC uses compiled HLSL shaders stored as CSO files. These have the magic DXBC in them, meaning they are `DirectX Bytecode`. We can use shader decompilation to convert the `cso`into `asm`, then `asm` to `hlsl` via 3dmigoto. Finally, I convert the `hlsl` into the Unreal cross-compiler-compatible format `usf`. I only use the pixel shader for now as the vertex shader is very complicated.

These `usf` files have their contents copied into the [custom shader material node](https://docs.unrealengine.com/4.27/en-US/RenderingAndGraphics/Materials/ExpressionReference/Custom/). In reality these HLSL shaders generate render target outputs of RT0, RT1, and RT2. I convert these RTs using information provided by Bungie in [talks](https://ubm-twvideo01.s3.amazonaws.com/o1/vault/gdc2018/presentations/Haraux_Alexis_Hawbaker_Nate_Translating_Art_Into_Technology.pdf). This allows for quite accurate shader recreation with very little manual effort.

## What can be improved?

Lots of things, but two things in particular. 
The first is the use of HLODs instead of static meshes - they're an optimisation technique in UE5 that allows for instancing of models. This could massively increase performance including with nanite in use.
The second is fixing the inaccuracies of the shader conversion. The HLSL -> USF decompilation and conversion is not perfect and introduces many assumptions. The most prominent of these is the `Normals` of the shader, which are very often incorrect. This is due to the game rendering out the world normal, where as unreal wants the texture normal among other issues. This might require the vertex shader for correct data to be provided.

You can help to fix these by [contributing to Charm](Development).