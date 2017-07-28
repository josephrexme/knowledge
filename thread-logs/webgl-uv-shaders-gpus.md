# 28-Jul-2017
shshaw [10:01 PM]
There’s no way to just rotate a texture in THREE, right?



----- Today July 28th, 2017 -----
mamboleoo [1:03 AM]
I don't think so, you'll have to draw your texture in a canvas then rotate it


visiblecode [1:35 AM]
Hmm, why not?


[1:37]
You can apply a transformation matrix to the texture coords in a shader before using them


[1:38]
If you want to "bake" the rotated texture as a texture for some reason, you can render a quad with the texture on to a render buffer set up to render to another texture


[1:39]
But usually you shouldn't need to do that


[1:41]
Transforming the texture coords during rendering should work better for most cases


shshaw [7:07 AM]
How do you transform the texture? Didn't see anything related to that in the docs


mamboleoo [7:09 AM]
From what I understood of @visiblecode message, you can apply a transform on the texture via a shader


[7:09]
This means using this material : https://threejs.org/docs/#api/materials/ShaderMaterial


nexii [7:35 AM]
Guess you’d have to actually rotate the `geometry` UV coords manually (for one-off rotation). (edited)


[7:37]
Shader would be probably the best choice if you wanted to change rotation frequently (edited)


nexii [7:42 AM]
Re-rendering the texture on a quad in the GPU would be a sort of middle solution (but don’t think the memory tradeoff is generally worth it imho, plus doesn’t scale well if you need multiple unique rotations)


shshaw [10:54 AM]
I just needed a one-time rotation, so I did it with canvas.



[10:54]
Unfortunate that it’s such a pain


visiblecode [12:19 PM]
You'll get better visual quality rotating UV coords instead of the image


shshaw [12:25 PM]
Can you explain UV cords or point me to what you mean? Is that part of the geometry, or the texture?


visiblecode [12:32 PM]
It's derived from the geometry by your shaders



visiblecode [12:38 PM]
I'm sort of fuzzy on where you're starting from re: WebGL experience but the basic idea is this:


[12:40]
Usually, you include a per-vertex vec2d attribute in your geometry data, which is used to determine what texture pixel is supposed to be lined up with that vertex


[12:42]
So if you're rendering a textured quad, the top left vertex might have this attribute as (0,0), the top right (1,0), the bottom left (0,1), the bottom right (1,1) (edited)


[12:44]
Which would map the whole texture onto the face of the quad without additional transformation besides the quad being in perspective or whatever


[12:45]
If you mess with those attribute values, you can stretch, rotate, etc, the texture as it appears on the face of the quad


[12:46]
If you want a fixed rotation for the texture, you might bake the rotation into the values you give for that attribute from the start (edited)


[12:47]
But you could also apply the rotation to those coords in your vertex shader (or fragment shader)


[12:51]
Unlike pre-rendering the rotated texture via canvas, manipulating the texture coordinates is output-resolution-independent, and it doesn't add extra processing work


visiblecode [12:57 PM]
To translate this to simple THREE terms

[12:57]
A vec2d is a THREE.Vector2

[12:58]
And the attribute array is .faceVertexUvs on your geometry object


[12:59]
By default THREE populates that with the (0,0), (1,0) etc kind of thing I talked about


shshaw [12:59 PM]
Ah, okay, so it’s a property of the geometry that determines how the material is painted on that geometry.


visiblecode [12:59 PM]
Yes


shshaw [1:00 PM]
Except instead of “painted on that geometry” its “how the pixels are drawn on the screen based on the geometry”,


[1:00]
Right?


visiblecode [1:00 PM]
Kind of


[1:01]
Thinking of it in terms of screen space might be confusing


[1:02]
Better to think of the UVs as "if I took the geometry and laid it on top of the texture, where would I need to put the vertexes to get the part of the texture I want to appear on the geometry"


[1:06]
And that's what the GPU does, for each screen pixel it refers to the UVs from the matching part of the geometry, and then finds the corresponding point in the texture (edited)


[1:06]
To get the color


shshaw [1:07 PM]
I see. I think it makes sense


visiblecode [1:10 PM]
(In reality, rather than consulting the geometry for each pixel there's a per-polygon division of labor between the vertex shader and the fragment shader, but I don't think that's super important to understand yet)


visiblecode [1:16 PM]
One other thing to be aware of is that in THREE, .faceVertexUVs is broken up by face


[1:17]
So it's indexed like faceVertexUvs[material][face][vertex]


[1:18]
And the vertex index is per-face, it's different from the index you use for .vertices


[1:19]
Which is different to how you'd normally do it in raw WebGL
