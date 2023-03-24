# SOEM_for_ROS2

## introduction

A slightly modified version of soem_ros2 that excludes warning that occurs when colcon build is executed，Original author project address：[Bitbucket](https://bitbucket.org/edhage/soem_ros2/src/master/)

## Building the package

1.Clone the git under ${WORKSPACE}/src

2.Go to workspace and colcon build

```shell
git clone 
cd ${WORKSPACE}
colcon build
```

## Usage

To use `soem` in your ROS2 package add the following to your `package.xml`and `CMakeLists.txt`, respectively.

In your `package.xml` add:

```xml
<depend>soem_ros</depend>
```

and in your `CMakeLists.txt`, add it to `find_package` as shown:

```cmake
find_package(soem_ros REQUIRED)
```

In the code, you just need to:

```c
#include "soem_ros/soem.h"
```

