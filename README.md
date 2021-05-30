# gcc-project-builder

gcc-project-builder provides a build system based on Makefiles containing standard recipes to build C/C++ projects.

For details, check [official repository](https://github.com/ljbo82/gcc-project-builder).

## Summary

* [License](#license)
* [Cloning](#cloning)
* [Usage](#usage)
* [Source directories](#source-directories)
* [Output directories](#output-directories)
* [Hosts](#hosts)
* [Input variables](#input-variables)
  * [Common input variables](#common-input-variables)
  * [Advanced input variables](#advanced-input-variables)
* [Recipes](#recipes)
* [Utility makefiles](#utility-makefiles)
  * [doxygen.mk](#doxygenmk)
  * [functions.mk](#functionsmk)
  * [git.mk](#gitmk)
  * [native_host.mk](#native_hostmk)

## License

gcc-project-builder is distributed under version 2 of the General Public License. Please see the LICENSE file for details on copying and distribution.

## Cloning

This repository uses [git-lfs](https://git-lfs.github.com/) to version some binaries files (for instance: images in [**doc/**](doc) directory). Ensure extension is installed before cloning this repository.

## Usage

gcc-project-builder provides a build system intended to be used by C/C++ projects in order to build its source files (C/C++/Assembly) using a GCC-based compiler.

Basically, the usage comprises the following steps:

1. Source files must be located in specific directories
2. Project's Makefile must declare some variables, which are used by the build system to generate an artifact (executable or library)
3. Project's Makefile includes **project.mk**
4. call `make`

Here is an example of a minimal Makefile used to build an executable with sources contained in project's **src/** directory:

```Makefile
PROJ_NAME := hello
PROJ_TYPE := app

include gcc-project-builder/project.mk
```

With this minimal Makefile, an executable can be build just by calling `make`.

For more examples, check the **demo/** directory.

## Source directories

When present, these directories (relative to projet Makefile) are used with the following purposes:

* **src/**

  Contains source files and private headers used by application during build. Any kind of file can be placed into this directory, but only files with standard filename extensions are compiled (case-sensitive): **.c** (C source file), **.cpp** (C++ source file), and **.S** (Assembly source file). This directory is also added to compiler's include search path.
  
  Additional source directories can be added to the project through the [`SRC_DIRS`](#var-src-dirs) [input variable](#input-variables).

* **include/**

  Contains public includes (header files) used by application during build. If project is a library, header files inside this directory will be copied to [distribution directory](#dir-dist). Any kind of file can be placed into this directory, but no compilation will be performed at all. This directory is added to compiler's include search path.
  
  Additional include directories can be added to the project through [`INCLUDE_DIRS`](#var-include-dirs) and [`DIST_INCLUDE_DIRS`](#var-dist-include-dirs) [input variables](#input-variables).

## Output directories

gcc-project-builder is inteded to support both native and cross-compilation. During build, output files are placed into host-specific directories (these output directories can be customized through [input variables](#input-variables):

<a name="dir-build"></a>
* **build/&lt;host>/**

  Build directory. Object files as well as final artifact (application executable or library) are placed into this directory. The build directory can be changed through [`BUILD_DIR_BASE`](#var-build-dir-base) and [`BUILD_DIR`](#var-build-dir) [input variables](#input-variables).

<a name="dir-dist"></a>
* **dist/&lt;host>/**

  Distribution directory. Final artifact (and possibly companion header, for libraries) are placed into this directory. Distribution directory can be changed through [`DIST_DIR_BASE`](#var-dist-dir-base) and [`DIST_DIR`](#var-dist-dir) [input variables](#input-variables). Additional directories containing companion headers to be distribuited along with library binary can be added through [`DIST_INCLUDE_DIRS`](#var-dist-include-dirs) [input variable](#input-variables).

## Hosts

TBD

## Input variables

The build system provided by **project.mk** can be customized through input variables.

Variable declaration and usage follows Makefile standard syntax.

All input variables must be declared/defined prior to **project.mk** inclusion.

Although it is perfectly legal to declare all input variables in a Makefile (or declaring all of them as environment variables), it its usually better to declare some of them in the Makefile and others as environment variables in order to achieve maximum build flexibility.

### Common input variables

Below are the list the commonly used input variables:

<a name="var-proj-name"></a>
* **`PROJ_NAME`**
  * Mandatory: **YES**
  * Common declaration: project's Makefile
  * Default value: _(not applicable)_

  Defines project name. It cannot contain spaces.

<a name="var-proj-type"></a>
* **`PROJ_TYPE`**
  * Mandatory: **YES**
  * Common declaration: project's Makefile
  * Default value: _(not applicable)_

  Defines project type. Accepted values are `app` (for an application) or `lib` (for a library).

<a name="var-lib-type"></a>
* **`LIB_TYPE`**
  * Mandatory: no
  * Common declaration: Either project's Makefile or environment
  * Default value: `shared`

  Defines library type. Accepted values are `shared` (for a shared library) or `static` (for a static library).

  For [`PROJ_TYPE`](#var-proj-type) other than `lib`, this variable is ignored.

<a name="var-proj-version"></a>
* **`PROJ_VERSION`**
  * Mandatory: no
  * Common declaration: Makefile
  * Default value: `0.1.0`

  Semantic version (`<major>.<minor>.<patch>`) for the project.

<a name="var-debug"></a>
* **`DEBUG`**
  * Mandatory: no
  * Common declaration: Environment
  * Default value: `0`

  Defines if a binary with debugging symbols shall be built. Accepted values are `0` (generates a binary WITHOUT debugging info) and `1` (generates an artifact WITH debug symbols).

<a name="var-v"></a>
* **`V`**
  * Mandatory: no
  * Common declaration: Environment
  * Default value: `0`

  Defines if `make` call shall output verbose information for build process. Accepted values are `0` (no verbosity) and `1` (verbose output).

<a name="var-host"></a>
* **`HOST`**
  * Mandatory: no
  * Common declaration: Environment
  * Default value: _(native host)_

  Sets the name which identifies the host for build artifacts (used for cross-compiling). 

  The value must follow the syntax `<os>-<arch>`. For more details, see [hosts](#hosts) section.

<a name="var-include-dirs"></a>
* **`INCLUDE_DIRS`**
  * Mandatory: no
  * Common declaration: Makefile
  * Default value: _(empty)_

  Defines extra include directories to be evaluated during build.

<a name="var-src-dirs"></a>
* **`SRC_DIRS`**
  * Mandatory: no
  * Common declaration: Makefile
  * Default value: _(empty)_

  Defines extra source directories containing files to be compiled. Paths cannot contain '..' substring.

<a name="var-cross-compile"></a>
* **`CROSS_COMPILE`**
  * Mandatory: no
  * Common declaration: Either project's Makefile or environment
  * Default value: _(empty)_

  Defines GCC prefix used for cross-compiling.

<a name="var-cflags"></a>
* **`CFLAGS`**
  * Mandatory: no
  * Common declaration: Makefile
  * Default value: _(empty)_

  Extra flags to be passed to C compiler.

<a name="var-cxxflags"></a>
* **`CXXFLAGS`**
  * Mandatory: no
  * Common declaration: Makefile
  * Default value: _(empty)_

  Extra flags to be passed to C++ compiler.

<a name="var-asflags"></a>
* **`ASFLAGS`**
  * Mandatory: no
  * Common declaration: Makefile
  * Default value: _(empty)_

  Extra flags to be passed to assembler.

<a name="var-ldflags"></a>
* **`LDFLAGS`**
  * Mandatory: no
  * Common declaration: Makefile
  * Default value: _(empty)_

  Extra flags to be passed to the linker. 

  NOTE: Linker executable will be `$(CROSS_COMPILE)gcc` (if project contains only C source files) or `$(CROSS_COMPILE)g++` (if project containts C++ source files).

<a name="var-prebuild"></a>
* **`PRE_BUILD`**
  * Mandatory: no
  * Common declaration: Makefile
  * Default value: _(empty)_

  Commands to be executed during [pre-build](#recipe-pre-build) [recipe](#recipes).

<a name="var-prebuild-deps"></a>
* **`PRE_BUILD_DEPS`**
  * Mandatory: no
  * Common declaration: Makefile
  * Default value: _(empty)_

  Project-specific depdencies for [pre-build](#recipe-pre-build) [recipe](#recipes).

<a name="var-build-deps"></a>
* **`BUILD_DEPS`**
  * Mandatory: no
  * Common declaration: Makefile
  * Default value: _(empty)_

  Project-specific dependencies for [build](#recipe-build) [recipe](#recipes).

<a name="var-post-build"></a>
* **`POST_BUILD`**
  * Mandatory: no
  * Common declaration: Makefile
  * Default value: _(empty)_

  Commands to be executed during [post-build](#recipe-post-build) [recipe](#recipes).

<a name="var-post-build-deps"></a>
* **`POST_BUILD_DEPS`**
  * Mandatory: no
  * Common declaration: Makefile
  * Default value: _(empty)_

  Project-specific dependencies for [post-build](#recipe-post-build) [recipe](#recipes).

<a name="var-pre-clean"></a>
* **`PRE_CLEAN`**
  * Mandatory: no
  * Common declaration: Makefile
  * Default value: _(empty)_

  Commands to be executed during [pre-clean](#recipe-pre-clean) [recipe](#recipes).

<a name="var-post-clean"></a>
* **`POST_CLEAN`**
  * Mandatory: no
  * Common declaration: Makefile
  * Default value: _(empty)_

  Commands to be executed during [post-clean](#recipe-post-clean) [recipe](#recipes).

<a name="var-pre-dist"></a>
* **`PRE_DIST`**
  * Mandatory: no
  * Common declaration: Makefile
  * Default value: _(empty)_

  Commands to be executed during [pre-dist](#recipe-pre-dist) [recipe](#recipes).

<a name="var-pre-dist-deps"></a>
* **`PRE_DIST_DEPS`**
  * Mandatory: no
  * Common declaration: Makefile
  * Default value: _(empty)_

  Project-specific dependencies for [pre-dist](#recipe-pre-dist) [recipe](#recipes).

<a name="var-dist-deps"></a>
* **`DIST_DEPS`**
  * Mandatory: no
  * Common declaration: Makefile
  * Default value: _(empty)_

  Project-specific dependencies for [dist](#recipe-dist) [recipe](#recipes).

<a name="var-post-dist"></a>
* **`POST_DIST`**
  * Mandatory: no
  * Common declaration: Makefile
  * Default value: _(empty)_

  Commands to be executed during [post-dist](#recipe-post-dist) [recipe](#recipes).

<a name="var-post-dist-deps"></a>
* **`POST_DIST_DEPS`**
  * Mandatory: no
  * Common declaration: Makefile
  * Default value: _(empty)_

  Project-specific dependencies for [post-dist](#recipe-post-dist) [recipe](#recipes).

### Advanced input variables

Below are the list the input variables for advanced usage:

<a name="var-hosts-dir"></a>
* **`HOSTS_DIR`**
  * Mandatory: no
  * Common declaration: Makefile
  * Default value: `hosts`

  Defines the directory (relative to project's Makefile) which contains host-specific makefile to be included (if present).

  This variable is useful for projects supporting multiple hosts with host-specific code and/or building procedure.

<a name="var-host-mk"></a>
* **`HOST_MK`**
  * Mandatory: no
  * Common declaration: Makefile
  * Default value: _(depends on selected [`$(HOST)`](#var-host))_

  This variable defines the name of the file inside [`$(HOSTS_DIR)`](#var-hosts-dir) which must be included for selected [`$(HOST)`](#var-host).

  For almost 100% of use-cases, default value is enough. For details regarding default behavior, see [hosts](#hosts) section.

  NOTE: Changing this variable is rarely required. Changing this variable is really an advanced topic and is only done when creating a custom build system on top of gcc-project-builder (e.g.: [arduino-gcc-project-builder](https://github.com/ljbo82/arduino-gcc-project-builder)).

<a name="var-host-mk-required"></a>
* **`HOST_MK_REQUIRED`**
  * Mandatory: no
  * Common declaration: Makefile
  * Default value: `0`

  This variable defines if [`$(HOSTS_DIR)`](#var-hosts-dir)/[`$(HOST_MK)`](#var-host-mk) must exist in order to build proceed.

  This variable is useful for projects targeting only specific hosts. When enabled (if its value is `1`), a valid hosts makefile must exist, otherwise an error will be raised informing that selected host is not supported.

<a name="var-builder-host-mk"></a>
* **`BUILDER_HOST_MK`**
  * Mandatory: no
  * Common declaration: Makefile
  * Default value: _(depends on selected [`$(HOST)`](#var-host))_

  This variable defines the name of host-specific makefile included by builder which contains instructions how to build artifacts.

  For almost 100% of use-cases, default value is enough. For details regarding default behavior, see [hosts](#hosts) section.

  NOTE: Changing this variable is rarely required. Changing this variable is really an advanced topic and is usually performed when creating a custom build system on top of gcc-project-builder (e.g.: [arduino-gcc-project-builder](https://github.com/ljbo82/arduino-gcc-project-builder)).

<a name="var-dist-include-dirs"></a>
* **`DIST_INCLUDE_DIRS`**
  * Mandatory: no
  * Common declaration: Makefile
  * Default value: _(empty)_

  This variable defines additional directories containing header files which must be copied to distribution package. Specified directories are also added to compiler include search path. 

<a name="var-build-dir-base"></a>
* **`BUILD_DIR_BASE`**
  * Mandatory: no
  * Common declaration: Environment
  * Default value: `build`

  Sets the name of the base directory (relative to project Makefile directory) where all build artifacts will be placed.

<a name="var-build-dir"></a>
* **`BUILD_DIR`**
  * Mandatory: no
  * Common declaration: Environment
  * Default value: `$(HOST)`

  Sets the name of the directory (relative to [`$(BUILD_DIR_BASE)`](#var-build-dir-base) where all build artifacts will be placed.

<a name="var-dist-dir-base"></a>
* **`DIST_DIR_BASE`**
  * Mandatory: no
  * Common declaration: Environment
  * Default value: `dist`

  Sets the name of the base directory (relative to project Makefile directory) where all distribution artifacts will be placed.

<a name="var-dist-dir"></a>
* **`DIST_DIR`**
  * Mandatory: no
  * Common declaration: Environment
  * Default value: `$(HOST)`

  Sets the name of the directory (relative to [`$(DIST_DIR_BASE)`](#var-dist-dir-base)) where all distribution artifacts will be placed.

<a name="var-as"></a>
* **`AS`**
  * Mandatory: no
  * Common declaration: Environment or Makefile
  * Default value: `as`

  Sets the name of native assembler executable.

<a name="var-ar"></a>
* **`AR`**
  * Mandatory: no
  * Common declaration: Environment or Makefile
  * Default value: `as`

  Sets the name of native archiver executable.

## Recipes

**project.mk** provides standard recipes used for almost all kind of C/C++ projects:

![Recipes overview](recipes-overview.png)

* **all**

  Default goal. Just depends on [**dist**](#recipe-dist) recipe.

* **build**

  Build project target artifact. It will be preceeded by [**pre-build**](#recipe-pre-build) recipe, dependencies declared in [`BUILD_DEPS`](#var-build-deps). It will also be followed by [**post-build**](#recipe-post-build).


## Utility makefiles

Along with **project.mk**, gcc-project-builder provides some utility makefiles which can be included to your project's Makefile in order to add more recipes, exposing functions, and exporting output variables, and so on.

### doxygen.mk

TBD

### functions.mk

TBD


### functions.mk

TBD

### git.mk

TBD


### native_host.mk

TBD

