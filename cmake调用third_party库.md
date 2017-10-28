# Document

[Cmake] <https://cmake.org/cmake/help/v3.8/manual/cmake-buildsystem.7.html>

[Qt官方手册] <http://doc.qt.io/qt-5/cmake-manual.html>

**Cmake 中调用Qt库**

```
cmake_minimum_required(VERSION 3.8)
project(untitled2)
set(CMAKE_PREFIX_PATH  /usr/local/Cellar/qt/5.9.0)
# Find includes in corresponding build directories
set(CMAKE_INCLUDE_CURRENT_DIR ON)
# Instruct CMake to run moc automatically when needed.
set(CMAKE_AUTOMOC ON)

# Find the QtWidgets library
find_package(Qt5Widgets)

# Tell CMake to create the helloworld executable
add_executable(helloworld  main.cpp)

# Use the Widgets module from Qt 5.
target_link_libraries(helloworld Qt5::Widgets)
```

**Cmake 中调用SDL2**

```
INCLUDE_DIRECTORIES(/Library/Frameworks/SDL2.framework/Headers/
        /Library/Frameworks/SDL2_image.framework/Headers/)
.......
TARGET_LINK_LIBRARIES(untitled5 ${SDL2_LIBRARIES} ${SDL2IMAGE_LIBRARY})
```

**Cmake中调用ffmpeg**

```
INCLUDE_DIRECTORIES(/usr/local/Cellar/ffmpeg/3.3.2/include)
link_directories(/usr/local/Cellar/ffmpeg/3.3.2/lib)
LINK_LIBRARIES(
        libavcodec.a    libavfilter.a   libavresample.a libpostproc.a   libswscale.a
        libavdevice.a   libavformat.a   libavutil.a     libswresample.a
)
```

**Cmake中调用opencv3**

```
set(OpenCV_DIR /usr/local/Cellar/opencv3/3.2.0/share/OpenCV)
find_package( OpenCV REQUIRED )
......
target_link_libraries( untitled5 ${OpenCV_LIBS})
```