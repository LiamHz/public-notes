Fundamentals of the Vulkan Graphics API
=======================================

## Introduction
Vulkan is a graphics API that is designed to provide an accurate abstraction of how modern graphics cards work. Unlike OpenGL, Vulkan is very verbose. Every detail related to the graphics API needs to be set up from scratch. The upside to this is that you can better understand what's going on in your application and in doing so achieve higher performance.

This article is intended to be a concise overview of the fundamentals of Vulkan. It was written for somebody who knew Vulkan, but has forgotten many of the details (i.e. future me). Most of the information here comes from the [Vulkan 1.2 Spec][001]. 

This article is not intended to be a follow along tutorial. You won't have to write code to get anything out of this article, but you will (hopefully) have a better understanding of how Vulkan and graphics processing units (GPUs) work.

Note: This article uses the [C++ bindings][005] for Vulkan

## Vulkan in a Nutshell
_From a bird's eye view, here's how a Vulkan application works:_

Vulkan can access **devices**, which let you control one or more **queues**. Queues are how you send a list of commands to the GPU, and they can be from one of several different **queue families**, each of which can do different things (e.g. drawing the vertices of a 3D model).

**Command buffers** are how you submit commands to a queue. Device commands are 'recorded' to a command buffer through Vulkan API calls, and can then be submitted once or many times (e.g. once every frame) to a queue to be executed. 

That's the TL;DR - now it's time to dive into the details!

## Genesis: Instances and Devices
In the beginning there was the Vulkan API. And the API was made concrete through `vk::Instance`, and from it came all per-application state.

Application state includes information like what version of the Vulkan API you're using, your application's name, and which extensions and layers you want to enable. Extensions and layers provide behavior that isn't included by default in Vulkan, like extended error checking and call logging.

With an instance you can examine the **physical devices** (usually GPUs) that are available. A machine might have multiple physical devices, and each of their properties / features (such as being a dedicated graphics card) can be inspected.

A common pattern for physical device selection is to:
- List all of the physical devices
- Score each physical device on desired properties
- Give physical devices without required properties a score of 0
- Pick the physical device with the highest score

### <!-- Example code for physical device selection -->
<details>
<summary><b>
Example code for physical device selection

(Tip: Read the comments so you don't get overwhelmed by code)
</b></summary>

```cpp
void pickPhysicalDevice() {
  // Get list of all physical devices that can be found by Vulkan
  auto physicalDevices = instance.enumeratePhysicalDevices();

  if (physicalDevices.size() == 0) {
    throw std::runtime_error("failed to find GPUs with Vulkan support!");
  }

  // Create list of physical devices sorted by rateDeviceSuitability()
  std::multimap<int, vk::PhysicalDevice> candidates;

  for (const auto& physicalDevice : physicalDevices) {
    int score = rateDeviceSuitability(physicalDevice);
    candidates.insert(std::make_pair(score, physicalDevice));
  }

  // Check if best candidate meets required properties (score > 0)
  if (candidates.rbegin()->first > 0) {
    physicalDevice = candidates.rbegin()->second;
  } else {
    throw std::runtime_error("failed to find a suitable GPU!");
  }
}

int rateDeviceSuitability(vk::PhysicalDevice physicalDevice) {
  // Get all features / properties of a given physical device
  vk::PhysicalDeviceFeatures deviceFeatures = physicalDevice.getFeatures();
  vk::PhysicalDeviceProperties deviceProperties = physicalDevice.getProperties();

  int score = 0;

  if (deviceProperties.deviceType == vk::PhysicalDeviceType::eDiscreteGpu) {
    score += 1000;  // Prefer dedicated GPUs
  }

  if (!deviceFeatures.geometryShader) {
    return 0;       // Require geometry shaders
  }

}
```
</details>

### <!-- Continued -->
With a physical device you can create a **logical device**, `vk::Device`.

Physical devices represent the GPU, and logical devices are what you use to create resources and access **queues**.

Keeping the logical device separate from the physical device is also useful because it allows you to have multiple logical devices, each with their state and resources, for a single physical device. One example of where this would be useful is if you had a Vulkan rendering application that also used a UI Toolkit (also running on Vulkan) that was completely separate. They'd each need their own logical devices to operate, but they're both using the same physical device.

Logical device creation is also where you create queues.

## Making Stuff Happen: Command Buffers and Queues
A queue, `vk::Queue` is a list of commands that the GPU executes.

Each queue can only have certain types of commands (some can have multiple types, others just one), and this is specified when a queue is created.

The four types of queue operations are:
- **Graphics** ---> Drawing the vertices of a model
- **Compute** ---> Ray tracing, cloth simulation
- **Transfer** ---> Loading textures and buffers
- **Sparse** ---> Loading part of a 'mega-texture'

A physical device will give you access to several queues of different **queue families**, and when you create a queue it will be an index to a queue of a matching queue family on the device. Queue families are queues that have the same properties as one another (e.g. perform both graphics and compute operations).

Commands are submitted to queues by first recording a series of commands into a command buffer, `vk::CommandBuffer`, and then submitting the entire command buffer to a queue with `vk::Queue::submit()`.

Note: Ideally you should reuse command buffers, but generally you can re-record them every frame without a large performance hit, and it makes some things easier (e.g. sending uniform buffers to shaders).

Also, you can submit multiple command buffers to a single queue. One reason why this is useful is because it _can_ allow you to ensure that one set of commands from a command buffer completes execution before another command buffer in the queue starts (more on this later).

Finally, commands recorded in command buffers can perform:
- **Actions** ---> draw, dispatch, clear, copy, query operations etc
- **State setting** ---> bind pipelines / buffers, push constants etc
- **Synchronization** ---> set / wait events, pipeline barriers etc

Some commands perform just one of these tasks, while others do several.

### <!-- Example code for command buffer creation -->
<details>
<summary><b>
Example code for command buffer creation
</summary></b>

```cpp
// Create a command buffer
vk::CommandBuffer cmd;

// Start recording to the command buffer
cmd.begin(vk::CommandBufferBeginInfo());

// Bind a graphics pipeline (covered later)
cmd.bindPipeline(vk::PipelineBindPoint::eGraphics, graphicsPipeline);

// Bind vertex buffers
cmd.bindVertexBuffers(0, 1, vertexBuffers, offsets);

// Draw the buffer bound to this command buffer
cmd.draw(vertices.size(), 1, 0, 0);

// Stop recording to this command buffer
cmd.end();
```
</details>

### <!-- Continued -->
Some commands that perform actions (e.g. draw / dispatch) do so based on the current state set by commands since the start of the command buffer. This means that in the above code example, `cmd.draw()` will operate on the current state set by `cmd.bindVertex()` in the previous line. This "synchronization guarantee", that one command will finish executing before the next one starts executing, is usually not true.

## Synchronization
Be warned, with few exceptions commands - regardless of what command buffer / queue they're in - **_are not executed in the order they were recorded in._** At places the Vulkan spec is confusing on this topic, but the TL;DR is that this is true unless a **synchronization object** is used.

Synchronization objects can either introduce **execution dependencies** or **memory dependencies**.

TODO Define execution dependencies

TODO Define memory dependencies

There are a few different types of synchronization objects
- Fences ---> GPU to CPU sync
- Semaphores ---> GPU to GPU sync
- Events
- Barriers ---> Synch within a command buffer

One more related operation of note is `vk::Queue::waitIdle()`, which is the same as submitting a fence to a queue and waiting indefinitely for that fence to signal.

## Render Passes

A render pass, `vk::RenderPass`, represents a set of framebuffer attachments and phases of rendering using those attachments.
## Command Pools
A command pool, `vk::CommandPool`, is where command buffer memory is allocated from and owned by.

## Graphics Pipeline
### Overview
The graphics pipeline is what takes the meshes and textures of 3D models (along with other information) and turns them into pixels on your screen. Each stage of the graphics pipeline operates on the output of the previous stage.

There are two types of stages in the graphics pipeline: shaders, and fixed-functions.

**Shaders** are user-created programs that execute the same set of instructions each frame on every vertex / geometry primitive / fragment (depending on the type of shader). Shaders run on GPUs which are great at parallel computing tasks, such as applying the same lighting rule for every one of the 2 millions pixels on your screen or rotating a 3D model with thousands of vertices.

**Fixed-functions** complete operations that can be tweaked with parameters, but the way they work is predefined.

A simplified overview of the graphics pipeline consists of 7 stages:

1. **Input Assembler:** collects the raw vertex data from specified buffers. Optionally, an index buffer can be used to repeat certain elements without duplicating vertex data.

2. **Vertex Shader:** runs on every vertex and passes per-vertex data down the graphics pipeline. Usually applies transformations to vertices, and converts from model space to screen space.

3. **Tessellation Shader:** Optional. Runs on arrays of vertices ("patches") and subdivides them into smaller primitives.

4. **Geometry Shader:** Optional. Runs on every primitive (triangle, line, point) and can discard or output more primitives. This stage is often not used because its performance isn't great on most graphics cards.

5. **Rasterization:** discretizes the primitives into fragments. Fragments that fall outside the screen, and fragments that are behind other primitives are discarded. 

6. **Fragment Shader:** runs on every fragment and determines its color, depth value, and which framebuffer the fragment is written to. Often uses interpolated data from the vertex shader such as surface normals to apply lighting.

7. **Color Blending:** applies operations to mix different fragments that map to the same pixel in the framebuffer. Fragments can overwrite each other, or be mixed based on transparency.

### Shader Modules
Unlike OpenGL, shader code in Vulkan has to be in a bytecode format called SPIR-V, as opposed to human-readable syntax like GLSL.

The advantage of using of using a bytecode format is that the compilers to turn shader code into native GPU code are significantly less complex. This leads to SPIR-V shader code being more reliable across GPU vendors.

However, shaders are still commonly written in GLSL, and later compiled to SPIR-V using a tool called `glslc` (included in the Vulkan SDK). This step can be included in your CMake build process.

SPIR-V can be passed to the graphics pipeline by reading the bytecode, and then wrapping it in a `vk::ShaderModule` object, and assigning it to a specific pipeline stage through the `vk::PipelineShaderStageCreateInfo` struct.

### Fixed Functions

## Memory Allocation

## Resource Creation

## The Framebuffer

### Swapchain
A swapchain, `vk::SwapchainKHR`, is an abstraction for an array of presentable images. One image from a swapchain is displayed at a time, but multiple images can be queued for presentation.

TODO Explain double and triple buffering

## Window System Integration

## Window Surface
TODO Refer to spec and edit

Window surfaces represent an abstract surface that Vulkan can present rendered images to. This allows for the core Vulkan API to be platform agnostic.

When ID Software ported DOOM 2016 (which used Vulkan) from Windows to Linux, all they had to do was change the window surface, and [performance was the same][004].

Window surfaces are optional in Vulkan, which makes off-screen rendering easy.

Note: Off-screen rendering is where you render to a framebuffer that doesn't directly impact the visual output of your window. A common use of this is for post-processing effects, where the scene is rendered to a framebuffer, and pixel shaders are applied to modify the image and render it on the screen.

## Validation Layers
The Vulkan API is designed around having minimal driver overhead. Because of this there is very limited error checking in the API by default.

Validation layers are optional components that hook into function calls for better error checking and other operations such as:
- Checking the values of function parameters against the Vulkan specification (e.g. incorrect enumeration values)
- Tracking creation and destruction of objects to find resource leaks
- Tracing Vulkan calls for profiling and replaying

Validation layers can be disabled in release builds to increase performance.

## Sources
- [vulkan-tutorial][003]
- [Vulkan in 30 Minutes][002]
- [What is Actually a Queue Family in Vulkan - StackOverflow][006]

## <!-- Links -->
[001]: https://www.khronos.org/registry/vulkan/specs/1.2/html/vkspec.html
[002]: https://renderdoc.org/vulkan-in-30-minutes.html
[003]: https://vulkan-tutorial.com/
[004]: https://youtu.be/0R23npUCCnw?t=2745
[005]: https://github.com/KhronosGroup/Vulkan-Hpp/Vulkan 
[006]: https://stackoverflow.com/questions/55272626/what-is-actually-a-queue-family-in-vulkan
