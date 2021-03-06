# Debugging a Segfault

1. I'm getting a segfault sometimes, but now always, when I run my program.

2. I set the build type in CMake to `Debug` --> This compiles an unoptimized version of my program, but with readable symbol tables (e.g. I see `error in createBuffer` instead of `error in #7876`)

3. I enable the address sanitizer with the `-fsanitize=address` flag in CMake --> This makes the program run ~50% slower, but will catch segfaults and other memory errors

4. I compile and run the program, get a segfault, and see `AddressSanitizer: heap-buffer-overflow on address 0x60...` and below:
```
#0 0x107611149 in vk::Fence::operator bool() const vulkan.hpp:14978
#1 0x10760fc25 in Excal::Frame::drawFrame(...) frame.cpp:95
#2 0x107680f31 in main main.cpp:235
```

5. `heap-buffer-overflow` means that I'm using an array to reach after its allocation (e.g. `IndexError`), and see that this is happening at line 95 in `frame.cpp`, when I call `drawFrame` at line 235 in `main.cpp`

This is the code around line 95 in `frame.cpp`
```cpp
// Check if a previous frame is using this image
// (i.e. there is its fence to wait on)
if (imagesInFlight[imageIndex]) {
  device.waitForFences(1, &inFlightFences[currentFrame], VK_TRUE, UINT64_MAX);
}
```

This shows that `imageInFlight[imageIndex]` is what's producing the `heap-buffer-overflow`

6. Image index can go up to a value of 2 (accessing the third element).

`imagesInFlight` was initialized with `std::vector<vk::Fence> imagesInFlight(maxFramesInFlight);` (maxFramesInFlight is a constant that's set to 2 currently)

**To fix the segfault, I need to change maxFramesInFlight to 3, instead of 2**

**Note:** Image index will go up to 2 if triple buffering is supported by the swapchain, and up to 1 if only double buffering is supported. This is why the error only occasionally happened.
