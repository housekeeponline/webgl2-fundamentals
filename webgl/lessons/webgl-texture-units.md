Title: WebGL Texture Units
Description: What are texture units in WebGL?
TOC: Texture Units


This article is meant to try to give you a mental image
of how texture units are setup in WebGL. There is [a similar article on attributes](webgl-attributes.html).

As a prerequisite you probably want to read [How WebGL Works](webgl-how-it-works)
and [WebGL Shaders and GLSL](webgl-shaders-and-glsl.html)
as well as [WebGL Textures](webgl-3d-textures.html)

## Texture Units

In WebGL there are textures. Textures are 2D arrays of data you can pass to a shader.
In the shader you declare a *uniform sampler* like this

```glsl
uniform sampler2D someTexture;
```

But how does the shader know which texture to use for `someTexture`?

That's where texture units come in. Texture units are a **global array** of
references to textures. You can imagine if WebGL was written in JavaScript
it would have some global state that looks like this

```js
const gl = {
  activeTextureUnit: 0,
  textureUnits: [
    { TEXTURE_2D: null, TEXTURE_CUBE_MAP: null, TEXTURE_3D: null, TEXTURE_2D_ARRAY, null, },
    { TEXTURE_2D: null, TEXTURE_CUBE_MAP: null, TEXTURE_3D: null, TEXTURE_2D_ARRAY, null, },
    { TEXTURE_2D: null, TEXTURE_CUBE_MAP: null, TEXTURE_3D: null, TEXTURE_2D_ARRAY, null, },
    { TEXTURE_2D: null, TEXTURE_CUBE_MAP: null, TEXTURE_3D: null, TEXTURE_2D_ARRAY, null, },
    { TEXTURE_2D: null, TEXTURE_CUBE_MAP: null, TEXTURE_3D: null, TEXTURE_2D_ARRAY, null, },
    { TEXTURE_2D: null, TEXTURE_CUBE_MAP: null, TEXTURE_3D: null, TEXTURE_2D_ARRAY, null, },
    { TEXTURE_2D: null, TEXTURE_CUBE_MAP: null, TEXTURE_3D: null, TEXTURE_2D_ARRAY, null, },
    { TEXTURE_2D: null, TEXTURE_CUBE_MAP: null, TEXTURE_3D: null, TEXTURE_2D_ARRAY, null, },
    { TEXTURE_2D: null, TEXTURE_CUBE_MAP: null, TEXTURE_3D: null, TEXTURE_2D_ARRAY, null, },
  ];
}
```

You can see above `textureUnits` is an array`. You assign a texture to one of the *bind points* in that array
of texture units. Let's assign `ourTexture` to texture unit 5.

```js
// at init time
const indexOfTextureUnit = 5;
const ourTexture = gl.createTexture();
// insert code it init texture here.

...

// at render time
gl.activeTexture(gl.TEXTURE0 + indexOfTextureUnit);
gl.bindTexture(gl.TEXTURE_2D, ourTexture);
```

You then tell the shader which texture unit bound the texture to by calling 

```js
gl.uniform1i(someTextureUniformLocation, indexOfTextureUnit);
```

If `activeTexture` and `bindTexture` WebGL functions were implemented in JavaScript they'd look
something like:

```js
// PSEUDO CODE!!!
gl.activeTexture = function(unit) {
  gl.activeTextureUnit = unit - gl.TEXTURE0;  // convert to index
};

gl.bindTexture = function(target, texture) {
  const textureUnit = gl.textureUnits[gl.activeTextureUnit];
  textureUnit[target] = texture;
}:
```

You can even imagine how other texture functions work. They all take a `target`
like `gl.texImage2D(target, ...)` or `gl.texParameteri(target)`. Those would
be implemented something like

```js
// PSEUDO CODE!!!
gl.texImage2D = function(target, level, internalFormat, width, height, border, format, type, data) {
  const textureUnit = gl.textureUnits[gl.activeTextureUnit];
  const texture = textureUnit[target];
  texture.mips[level] = convertDataToInternalFormat(internalFormat, width, height, format, type, data);
}

gl.texParameteri = function(target, pname, value) {
  const textureUnit = gl.textureUnits[gl.activeTextureUnit];
  const texture = textureUnit[target];
  texture[pname] = value; 
}
```

## Maximum Texture Units

WebGL requires an implementation to support at least 32 texture units. You can query how many
are supported with

```js
const maxTextureUnits = gl.getParameter(gl.AX_COMBINED_TEXTURE_IMAGE_UNITS);
```

Note that vertex shaders and fragment shaders might have different limits
on how many units each can use. You can query the limits for each with

```js
const maxVertexShaderTextureUnits = gl.getParameter(gl.MAX_VERTEX_TEXTURE_IMAGE_UNITS);
const maxFragmentShaderTextureUnits = gl.getParameter(gl.MAX_TEXTURE_IMAGE_UNITS);
```

Each are required to support at least 16 texture unts.

Let's say 

```js
maxTextureUnits = 32
maxVertexShaderTextureUnits = 16
maxFragmentShaderTextureUnits = 32
```

This means if you use for example 2 texture units in your vertex shader
there are only 30 left to use in your fragment shader since the combined
maximum is 32.
