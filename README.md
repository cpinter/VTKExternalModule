VTKExternalModule
=================

[![Project Status: WIP – Initial development is in progress, but there has not yet been a stable, usable release suitable for the public.](https://www.repostatus.org/badges/latest/wip.svg)](https://www.repostatus.org/#wip)

CMake buildsystem for externally building any built-in or remote VTK module outside of the VTK source tree.

CMake options
-------------

_Selected options specific to this project. For a complete list of options, inspect the [CMakeLists.txt][CMakeLists] and check documentation specific to CMake and VTK._

[CMakeLists]: CMakeLists.txt


| Name | Description | Required |
|------|-------------|----------|
| `VTK_MODULE_SOURCE_DIR` | Path to directory containing a [vtk.module][vtk.module] file. | :heavy_check_mark: |
| `VTK_MODULE_NAME` | Name of the module specified after `VTK::`. | :heavy_check_mark: |
| `VTK_MODULE_CMAKE_MODULE_PATH` | Optional path to prepend to `CMAKE_MODULE_PATH`. | |

[vtk.module]: https://vtk.org/doc/nightly/html/group__module.html#module-overview

License
-------

It is covered by the Apache License, Version 2.0:

http://www.apache.org/licenses/LICENSE-2.0
