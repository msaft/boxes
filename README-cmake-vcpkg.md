# CMake and vcpkg based build environment

## Prerequisites

Building boxes requires libraries. To resolve these requirements we're using [vcpkg](https://vcpkg.io/), a package
manager for C/C++. Follow the [Get Started](https://vcpkg.io/en/getting-started) guide in case `vcpkg` isn't installed
yet. It is probably a good idea to use a released version, i.e.,

```sh
git clone https://github.com/Microsoft/vcpkg.git
cd vcpkg
git checkout -b b2023.10.19 2023.10.19
./bootstrap-vcpkg.sh -disableMetrics
```

Let's assume `vcpkg` is installed in `~/Applications/Developer/Tools/vcpkg`. Define the following environment variables:

```sh
export VCPKG_ROOT="${HOME}/Applications/Developer/Tools/vcpkg"
export PATH=${VCPKG_ROOT}:$PATH
```

For building boxes, we additionally need the following tools to be installed:

- `flex`
- `bison`

And for building the dependencies, we also need:

- `automake`
- `autoconf`
- `pkg-config`

Make sure that these tools are available at your command line.

## Initialize the project

We are using `vcpkg` in the so called [manifest mode](https://learn.microsoft.com/en-gb/vcpkg/concepts/manifest-mode).
Creating a manifest and adding dependencies to it can be done with the `vcpkg` command:

```sh
vcpkg new --application

vcpkg add port libunistring
vcpkg add port pcre2
vcpkg add port ncurses
```

Now we have two new files in our project root directory:

- `vcpkg.json`: Defines the required dependencies.
- `vcpkg-configuration.json`: Specifying repositories.

Finally, create a `CMakePresets.json` file so that we don't have to specify the path to `vcpkg` on the command line:

```json
{
  "version": 3,
  "configurePresets": [
    {
      "name": "default",
      "binaryDir": "${sourceDir}/build",
      "cacheVariables": {
        "CMAKE_TOOLCHAIN_FILE": "$env{VCPKG_ROOT}/scripts/buildsystems/vcpkg.cmake"
      }
    }
  ],
  "buildPresets": [
    {
      "name": "default",
      "configurePreset": "default"
    }
  ]
}
```

## Building boxes

```sh
cmake -DCMAKE_EXPORT_COMPILE_COMMANDS=ON --preset=default
cmake --build --preset=default

ctest --verbose --preset conan-debug
```
