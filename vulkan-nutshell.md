Vulkan in a Nutshell
====================

## Introduction
Vulkan is a graphics API that is designed to provide an accurate abstraction of how modern graphics cards work. Unlike OpenGL, Vulkan is very verbose. Every detail related to the graphics API needs to be manually set up from scratch. The upsides to this are that you can achieve better performance, and develop applications that are fully cross-platform.

This article is intended to be a quick recap of processes you need to understand to use Vulkan. It was written for somebody who knew Vulkan, but has forgotten many of the details (i.e. future me). All of the information here comes from the [Vulkan 1.2 Spec][001], [Vulkan in 30 minutes][002], and [vulkan-tutorial][003].

## Instance
A Vulkan instance, `VkInstance`, is the connection between an application and the Vulkan library. It stores per-application state, notably, what API extensions and validation layers will be enabled.

## Physical Device
A physical device, `VkPhysicalDevice`, represents a graphics card. Most Vulkan applications have preferred and required properties / features of the physical device they will use. A modern video game that uses Vulkan might require support for geometry shaders, and prefer a dedicated graphics card over an integrated one.

A common pattern for physical device selection is to:
- List all of the physical devices
- Score each device on its properties / features
- Give physical devices without required properties / features a score of 0
- Pick the device with the highest score

<details>
<summary><b>
Code for Physical Device Selection
</b></summary>

```cpp
#include <map>

void pickPhysicalDevice() {
  // Create a vector of all detected physical devices
  uint32_t deviceCount = 0;
  vkEnumeratePhysicalDevices(instance, &deviceCount, nullptr);
  std::vector<VkPhysicalDevice> devices(deviceCount);
  vkEnumeratePhysicalDevices(instance, &deviceCount, devices.data());

  // Create a map of physical devices sorted by their score
  std::multimap<int, VkPhysicalDevice> candidates;

  // Assign a score to each physical device
  for (const auto& device : devices) {
    int score = rateDeviceSuitability(device);
    candidates.insert(std::make_pair(score, device));
  }

  // A score of indicates that the best device isn't suitable
  if (candidates.rbegin()->first > 0) {
    physicalDevice = candidates.rbegin()->second;
  } else {
    throw std::runtime_error("failed to find a suitable GPU!");
  }
}

int rateDeviceSuitability(VkPhysicalDevice device) {
  // Get all features / properties of a given physical device
  VkPhysicalDeviceFeatures deviceFeatures;
  VkPhysicalDeviceProperties deviceProperties;
  vkGetPhysicalDeviceFeatures(device, &deviceFeatures);
  vkGetPhysicalDeviceProperties(device, &deviceProperties);

  int score = 0;
  if (deviceProperties.deviceType == VK_PHYSICAL_DEVICE_TYPE_DISCRETE_GPU) {
    score += 1000;  // Prefer dedicated GPUs
  }

  if (!deviceFeatures.geometryShader) {
    return 0;       // Require geometry shaders
  }

  return score;
}
```
</details>

## Logical Device
A logical device, `VkDevice`, is created from a `VkPhysicalDevice`, and it is what interfaces with a physical device.

`VkDevice` is used to create pretty much every Vulkan resource type (e.g. images and buffers) and is passed as an argument to many Vulkan functions, such as `vkCreateRenderPass` and `vkCreateGraphicsPipelines`.

## Queue Families
### Present Queue Family
### Graphics Queue Family

## Window Surface

## Swap Chain

## Image Views

## Framebuffers

## Render Passes

## Graphics Pipeline

## Command Pools

## Command Buffers

## Main Loop
### Acquire Next Image
### Frame
### Present Image

## Synchronization Objects

## Validation Layers
The Vulkan API is designed around having minimal driver overhead. Because of this there is very limited error checking in the API by default.

Validation layers are optional components that hook into function calls for better error checking and other operations such as:
- Checking the values of function parameters against the Vulkan specification (e.g. incorrect enumeration values)
- Tracking creation and destruction of objects to find resource leaks
- Tracing Vulkan calls for profiling and replaying

Validation layers can be disabled in release builds to increase performance.

## Misc
### Vulkan Coding Conventions
Vulkan API Prefixes
- vk: functions
- Vk: structs and enumerations
- VK_: enumeration values

Creating a Vulkan object typically involves passing large structs to functions, instead of many parameters.

**Pseudocode for Vulkan Object Creation**
```cpp
VkXXXCreateInfo createInfo{};
createInfo.sType = VK_STRUCTURE_TYPE_XXX_CREATE_INFO;
createInfo.foo = ...;
createInfo.bar = ...;

VkXXX object;
if (vkCreateXXX(&createInfo, nullptr, &object) != VK_SUCCESS) {
    throw std::runtime_error("failed to create object!");
}
```

The pattern here is that:
- A `createInfo` struct is created. Its member variables specify initialization info.
- `createInfo.sType` specifies the type of structure.
- `vkCreateXXX()` is called with references to `createInfo` and the object to be created.
- `vkCreateXXX()`'s second argument allows you to use a custom allocator for driver memory
- `vkCreateXXX()` returns either `VK_SUCCESS` or an error code

## <!-- Links -->
[001]: https://www.khronos.org/registry/vulkan/specs/1.2/html/vkspec.html
[002]: https://renderdoc.org/vulkan-in-30-minutes.html
[003]: https://vulkan-tutorial.com/
