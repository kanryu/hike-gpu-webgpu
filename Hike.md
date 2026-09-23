# Hike WebGPU Module

This directory is structured as the standalone Hike module published at:

<https://github.com/kanryu/hike-gpu-webgpu>

The intended import path is:

```hike
import "github.com/kanryu/hike-gpu-webgpu/webgpu"
```

The module exposes a small typed wrapper around a browser-provided WebGPU
bridge. `webgpu.NewContext` creates an opaque context for a canvas element,
resource helpers create buffers, shader modules, render pipelines, and bind
groups, and `Draw` submits a render pass. `Resize` and `Close` manage the
context lifecycle. The API uses integer handles, so applications do not need
to know the layout of browser-owned WebGPU objects.

The current convenience pipeline accepts a vertex buffer with position and
color attributes and a uniform buffer at binding zero. This is a reusable
baseline for small Hike rendering applications rather than an application-
specific triangle implementation; applications can provide their own WGSL
shader source and draw count.

Example usage:

```hike
import "github.com/kanryu/hike-gpu-webgpu/webgpu"

func InitApp() {
    context := webgpu.NewContext("gpu-canvas", 960, 640)
    vertexBuffer := webgpu.CreateBuffer(context, 72, 40)
    shader := webgpu.CreateShaderModule(context, shaderSource)
    pipeline := webgpu.CreateRenderPipeline(context, shader, 24)
    bindGroup := webgpu.CreateBindGroup(context, pipeline, uniformBuffer)
    webgpu.Draw(context, pipeline, bindGroup, vertexBuffer, 3)
}
```

## Dependency usage

After publishing this directory as its own repository, a Hike application can
declare the dependency in `hike.mod`:

```text
module my-project

hike 0.1.0

require github.com/kanryu/hike-gpu-webgpu v0.1.0
```

The planned `hike get` command will clone required Git modules into the local
Hike module cache. Builds then resolve the package by its module import path.
Until that command is available, development projects can use a local
replacement:

```text
replace github.com/kanryu/hike-gpu-webgpu => ../hike-gpu-webgpu
```

## Browser bridge

The package contains the `jfunc` browser bridge. It creates the adapter and
device, configures the canvas, queues resource creation until the device is
ready, uploads buffer data, and submits render passes. WebGPU objects remain
JavaScript-owned and are represented by opaque handles in Hike, keeping the
Wasm memory model stable and allowing the same module to be imported by
multiple applications.
