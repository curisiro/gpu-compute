# gpu-compute

## execute shader programs

[![Build Status](https://travis-ci.org/demskie/gpu-compute.svg?branch=master)](https://travis-ci.org/demskie/gpu-compute) [![Coverage Status](https://coveralls.io/repos/github/demskie/gpu-compute/badge.svg?branch=master)](https://coveralls.io/github/demskie/gpu-compute?branch=master)
[![Dependency Status](https://david-dm.org/demskie/gpu-compute/status.svg)](https://david-dm.org/demskie/gpu-compute#info=dependencies&view=table)

## Warning

```text
THIS LIBRARY IS EXPERIMENTAL!!!

NodeJS support is a work in progress
```

## Installation

```bash
npm install gpu-compute
```

## Example

```js
import * as gpgpu from "gpu-compute";

// Each texel is packed with two 16bit ints.
// This program continuously increments those values using floored coordinates.
var source = `
precision mediump float;
precision mediump int;
precision mediump sampler2D;

uniform sampler2D u_gpuData;
const float TEXTURE_WIDTH = ${textureWidth};

float vec2ToInt16(vec2 v) { return clamp(floor(floor(v.r * 255.0) * 256.0) + floor(v.g * 255.0) - 32767.0, -32767.0, 32768.0); }
vec2 int16ToVec2(float f) { f = clamp(f, -32767.0, 32768.0) + 32767.0; return vec2(floor(f / 256.0), f - floor(f / 256.0) * 256.0) / 255.0; }

void main() {
  vec4 texel = texture2D(u_gpuData, gl_FragCoord.xy / TEXTURE_WIDTH);
  float x = mod(vec2ToInt16(texel.rg) + floor(gl_FragCoord.x), TEXTURE_WIDTH);
  float y = mod(vec2ToInt16(texel.ba) + floor(gl_FragCoord.y), TEXTURE_WIDTH);
  gl_FragColor = vec4(int16ToVec2(x), int16toVec2(y));
}`

// initialize primatives
var target = new gpgpu.RenderTarget(128);
var shader = new gpgpu.ComputeShader(source, { textureWidth: "128.0" });

// push random data into texture
var allSevensArray = new Uint8Array(128 * 128 * 4).fill(7);
target.pushTextureData(allSevensArray);

// loop program using previous output as input
for (var i = 0; i < 4; i++) {
  target.compute(shader, { u_gpuData: target });
  console.log(target.readPixels());
}
```
