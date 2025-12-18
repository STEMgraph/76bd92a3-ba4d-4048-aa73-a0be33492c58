<!---
{
  "id": "76bd92a3-ba4d-4048-aa73-a0be33492c58",
  "depends_on": ["ba108c71-19c6-4063-b813-fdbbe0b5d775","722698f1-d030-4813-ab8a-b9fd0c92bf27"],
  "author": "Stephan Bökelmann",
  "first_used": "2025-12-18",
  "keywords": ["CMake", "C++", "FetchContent", "External Library", "Git", "modular programming"],
  "license": "   
Copyright 2025 Stephan Bökelmann

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License."
}
--->

# Using `FetchContent` in CMake to Include a Remote C++ Library

> In this exercise you will learn how to integrate remote C++ libraries using CMake's `FetchContent` module. Furthermore we will explore how to create and publish your own reusable library on GitHub, so that it can be included by others via `FetchContent`.

## Introduction

As C++ projects grow and diversify, using third-party and self-developed libraries becomes increasingly common. Traditionally, developers manually copied code or used package managers. With modern CMake (v3.11+), `FetchContent` offers a clean alternative to bring external repositories into your build at configure time, without needing global installs or extra tooling.

In this exercise, you will practice two essential developer workflows:

1. Using a remote GitHub-hosted library via `FetchContent`, and calling its functionality in a simple program.
2. Creating your **own** library, pushing it to GitHub, and reusing it in another project using `FetchContent`.

Both workflows strengthen your understanding of project modularization, version control, and cross-repository reuse. `FetchContent` pulls the repository at configuration time, builds it as part of your project, and handles include directories and targets automatically.

Unlike `add_subdirectory`, which assumes local folders, `FetchContent` reaches out to remote URLs, clones or updates the content, and integrates it like a native subproject. You don't even need to commit the external code to your repo. This approach mimics how modern toolchains handle dependencies (e.g., `vcpkg`, `Conan`, or `Bazel`).

This exercise prepares you for reusable, component-based C++ development.

### Further Readings and Other Sources

* [CMake FetchContent documentation](https://cmake.org/cmake/help/latest/module/FetchContent.html)
* YouTube: [Modern CMake — FetchContent Tutorial](https://www.youtube.com/watch?v=0q1M3T_QjYQ)
* GitHub Example: [STEMgraph/cmake_test_library](https://github.com/STEMgraph/cmake_test_library)
* Book: *Professional CMake: A Practical Guide* by Craig Scott (GitHub: [crascit](https://github.com/crascit))

---

## Tasks

---

# **Task 1 — Use an Existing Remote Library with `FetchContent`**

You will use the `STEMgraph/cmake_test_library` hosted on GitHub.

### 1. Create a new project

Create the following directory structure using `touch` and `mkdir`:

```
fetch_demo/
├── CMakeLists.txt
└── src/
    └── main.cpp
```

Create the main program:

**`src/main.cpp`**

```cpp
#include "hello.h"

int main() {
    STEMgraph::printHello();  // Should print: "Hello from CMake Fetch Content installed Library!"
    return 0;
}
```

### 2. Write your `CMakeLists.txt`

In the root of `fetch_demo/`:

```cmake
cmake_minimum_required(VERSION 3.16)
project(FetchContentDemo)

include(FetchContent)

FetchContent_Declare(
  stemgraph_cmake_lib
  GIT_REPOSITORY https://github.com/STEMgraph/cmake_test_library.git
  GIT_TAG master
)

FetchContent_MakeAvailable(stemgraph_cmake_lib)

add_executable(fetch_demo src/main.cpp)
target_link_libraries(fetch_demo PRIVATE STEMgraphCMakeLib)
```

### 3. Build and run

```bash
cmake -B build
cmake --build build
./build/fetch_demo
```

Expected output:

```
Hello from CMake Fetch Content installed Library!
```

---

# **Task 2 — Create Your Own GitHub Library and Use It Remotely**

### 1. Make a new GitHub repository

* Name it something like `my_cmake_library`
* Make it public
* Clone it to your local machine

### 2. Create the library structure

Inside your cloned repository, create the following directory structure:

```
my_cmake_library/
├── CMakeLists.txt
├── include/
│   └── mylib.h
└── src/
    └── mylib.cpp
```

### 3. Write the header and source files

**`include/mylib.h`**

```cpp
#pragma once

namespace MyLib {
    void greet();
}
```

**`src/mylib.cpp`**

```cpp
#include <iostream>
#include "mylib.h"

namespace MyLib {
    void greet() {
        std::cout << "Greetings from your self-published CMake library!" << std::endl;
    }
}
```

### 4. Add the library `CMakeLists.txt`

```cmake
cmake_minimum_required(VERSION 3.16)
project(MyCMakeLib)

add_library(${PROJECT_NAME} STATIC src/mylib.cpp)

target_include_directories(${PROJECT_NAME} PUBLIC include/)
```

Commit and push your changes:

```bash
git add .
git commit -m "Initial commit of self-written library"
git push origin main
```

### 5. Create a new project that uses your library via `FetchContent`

Create the following directory structure:

```
use_my_lib/
├── CMakeLists.txt
└── src/
    └── main.cpp
```

**`src/main.cpp`**

```cpp
#include "mylib.h"

int main() {
    MyLib::greet();  // Your function from the remote library
    return 0;
}
```

**`CMakeLists.txt`**

Replace the `GIT_REPOSITORY` URL with your actual GitHub repository:

```cmake
cmake_minimum_required(VERSION 3.16)
project(MyLibConsumer)

include(FetchContent)

FetchContent_Declare(
  mylib
  GIT_REPOSITORY https://github.com/<YOURUSERNAME>/my_cmake_library.git
  GIT_TAG main      # or master or a tag
)

FetchContent_MakeAvailable(mylib)

add_executable(${PROJECT_NAME} src/main.cpp)

# PRIVATE is used because my_app is the final executable and doesn't need to propagate the library's properties to other targets.
target_link_libraries(${PROJECT_NAME} PRIVATE MyCMakeLib)
```

Then build and run:

```bash
cmake -B build
cmake --build build
./build/my_app
```

Expected output:

```
Greetings from your self-published CMake library!
```

---

## Questions

**1. What is the difference between `FetchContent_Declare` and `add_subdirectory` in terms of dependency inclusion?**
<details>
<summary>Click to reveal answer</summary>

`add_subdirectory` requires the library to exist locally and within your source tree, while `FetchContent_Declare` can pull and build remote libraries at configuration time, making it ideal for third-party or shared libraries.

</details>

**2. Why is `target_include_directories(... PUBLIC ...)` required for your library?**
<details>
<summary>Click to reveal answer</summary>

Because it ensures that when another target links to the library, it inherits the include paths necessary to use the library's headers.
</details>

**3. What happens if your GitHub library doesn't contain a valid `CMakeLists.txt`?**
<details>
<summary>Click to reveal answer</summary>

The `FetchContent_MakeAvailable` step will fail because CMake cannot configure or build the dependency if it lacks a proper build definition.

</details>

---

## Advice

Using `FetchContent` can simplify development workflows, especially in early-stage projects or when you control the dependencies. It eliminates the need to clone and configure libraries manually. Once you publish your own library and use it remotely, you become a contributor to a modular ecosystem. Be sure to test your libraries in isolation, keep your public interfaces clean, and document expected usage. Reusability and maintainability are key. See also: [Exercise on `add_subdirectory`](https://github.com/STEMgraph/ba108c71-19c6-4063-b813-fdbbe0b5d775) for a local-library alternative.

---
