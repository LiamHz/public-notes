How to Install Vulkan on macOS
=============================

## Install Software
- Run `brew install glfw3 glm pkg-config cmake` to install dependencies
- Download the [Vulkan SDK][001] for macOS.
- Run `mkdir /usr/local/vulkansdk` and copy the contents of the SDK into the folder
- Run `vulkaninfo` to check that Vulkan can find your GPU
- In your terminal's profile (`~/.zprofile` if you use zsh, `~/.bash_profile` if you use bash) add the following lines to the end of the file, and then close and reopen your terminal
- Finally, run `vkvia` to run the Vulkan installation analyzer. If you see a rotating cube and your terminal doesn't output any error messages, you've installed Vulkan correctly!

**Lines to add to your terminal's profile**
```
export VULKAN_SDK="/usr/local/vulkansdk/macOS"
export VK_ICD_FILENAMES="$VULKAN_SDK/share/vulkan/icd.d/MoltenVK_icd.json"
export VK_LAYER_PATH="$VULKAN_SDK/share/vulkan/explicit_layer.d"
export DYLD_LIBRARY_PATH="$VULKAN_SDK/lib:${DYLD_LIBRARY_PATH:-}"
```

## Set up test project
- Create a directory for your test project and `cd` into it
- Create a source file called `main.cpp` and insert the following code

```cpp
#define GLFW_INCLUDE_VULKAN
#include <GLFW/glfw3.h>

#define GLM_FORCE_RADIANS
#define GLM_FORCE_DEPTH_ZERO_TO_ONE
#include <glm/vec4.hpp>
#include <glm/mat4x4.hpp>

#include <iostream>

int main() {
    glfwInit();

    glfwWindowHint(GLFW_CLIENT_API, GLFW_NO_API);
    GLFWwindow* window = glfwCreateWindow(800, 600, "Vulkan window", nullptr, nullptr);

    uint32_t extensionCount = 0;
    vkEnumerateInstanceExtensionProperties(nullptr, &extensionCount, nullptr);

    std::cout << extensionCount << " extensions supported\n";

    glm::mat4 matrix;
    glm::vec4 vec;
    auto test = matrix * vec;

    while(!glfwWindowShouldClose(window)) {
        glfwPollEvents();
    }

    glfwDestroyWindow(window);

    glfwTerminate();

    return 0;
}
```

- Create a file called `CMakeLists.txt` and insert the following code into it

```
project(vkTutorial)
cmake_minimum_required(VERSION 3.10)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -O3 -std=c++11  " )

add_executable(${PROJECT_NAME} main.cpp)

find_package(Vulkan REQUIRED)
find_package(glfw3 3.3 REQUIRED)

if (VULKAN_FOUND)
    message(STATUS "Found Vulkan, Including and Linking now")
    include_directories(${Vulkan_INCLUDE_DIRS})
	target_link_libraries (${PROJECT_NAME} ${Vulkan_LIBRARIES} glfw)
endif (VULKAN_FOUND)
```
- Run `mkdir build` and `cd` into it
- Run `cmake ..`, then `make`, and finally `open $EXECUTABLE_NAME`

If everything is set up correctly, an empty window should open the terminal should display the number of Vulkan extensions.

Now you have Vulkan up and running!

[001]: https://vulkan.lunarg.com/sdk/home
