# SOEM_for_ROS2

## introduction

A slightly modified version of soem_ros2 that excludes warning that occurs when colcon build is executed，Original author project [address](https://bitbucket.org/edhage/soem_ros2/src/master/).

## Fixed problem

#### Warning-1:

#### ![image-20230324162042165](https://github.com/evencewu/SOEM_for_ROS2/blob/main/png/image-20230324162042165.png)

#### resolvent：

```
message("phtread=${PTHREAD_LIB}")
```

Turn into

```
message(STATUS "phtread=${PTHREAD_LIB}")
```

#### Warning-2:

![image-20230324162613210](https://github.com/evencewu/SOEM_for_ROS2/blob/main/png/image-20230324162613210.png)

#### resolvent：

![image-20230324164322108](https://github.com/evencewu/SOEM_for_ROS2/blob/main/png/image-20230324164322108.png)

Turn into

![image-20230324164247035](https://github.com/evencewu/SOEM_for_ROS2/blob/main/png/image-20230324164247035.png)

The compiler does not seem to support nested anonymous structures in the union，so we change the anonymous structure and all the places it is used.

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

