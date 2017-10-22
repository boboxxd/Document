# Document


[官方手册] <http://doc.qt.io/qt-5/cmake-manual.html>
eg:
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
