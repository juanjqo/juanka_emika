cmake_minimum_required(VERSION 3.5)

project(JuankaEmika VERSION 0.1 LANGUAGES CXX)

set(CMAKE_INCLUDE_CURRENT_DIR ON)

set(CMAKE_AUTOUIC ON)
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)


if(UNIX AND NOT APPLE)
    FIND_PACKAGE(Eigen3 REQUIRED)
    INCLUDE_DIRECTORIES(${EIGEN3_INCLUDE_DIR})
    ADD_COMPILE_OPTIONS(-Werror=return-type -Wall -Wextra -Wmissing-declarations -Wredundant-decls -Woverloaded-virtual)
endif()

if(APPLE)
    INCLUDE_DIRECTORIES(
        /usr/local/include/
        /usr/local/include/eigen3
        # Most recent versions of brew install here
        /opt/homebrew/include
        /opt/homebrew/include/eigen3)
    ADD_COMPILE_OPTIONS(-Werror=return-type -Wall -Wextra -Wmissing-declarations -Wredundant-decls -Woverloaded-virtual)
    # The library is installed here when using the regular cmake ., make, sudo make install
    LINK_DIRECTORIES(
        /usr/local/lib/)
endif()

find_package(Franka REQUIRED)

find_package(yaml-cpp REQUIRED)

include_directories(${EIGEN3_INCLUDE_DIR})


include_directories(
    sas_robot_driver_franka/include
    sas_robot_driver_franka/constraints_manager/include
    )


add_library(MotionGenerator sas_robot_driver_franka/src/motion_generator.cpp)
target_link_libraries(MotionGenerator Franka::Franka)


add_library(ConstraintsManager sas_robot_driver_franka/constraints_manager/src/constraints_manager.cpp)

add_library(QuadraticProgramMotionGenerator sas_robot_driver_franka/src/quadratic_program_motion_generator.cpp)
target_link_libraries(QuadraticProgramMotionGenerator
                     qpOASES
                     dqrobotics
                     ConstraintsManager)



add_library(robot_interface_franka sas_robot_driver_franka/src/robot_interface_franka.cpp)
target_link_libraries(robot_interface_franka Franka::Franka
                    dqrobotics
                    MotionGenerator
                    QuadraticProgramMotionGenerator)


find_package(QT NAMES Qt6 Qt5 REQUIRED COMPONENTS Widgets)
find_package(Qt${QT_VERSION_MAJOR} REQUIRED COMPONENTS Widgets)

set(PROJECT_SOURCES
        src/main.cpp
        src/mainwindow.cpp
        src/mainwindow.h
        src/mainwindow.ui
)

if(${QT_VERSION_MAJOR} GREATER_EQUAL 6)
    qt_add_executable(JuankaEmika
        MANUAL_FINALIZATION
        ${PROJECT_SOURCES}
    )
# Define target properties for Android with Qt 6 as:
#    set_property(TARGET JuankaEmika APPEND PROPERTY QT_ANDROID_PACKAGE_SOURCE_DIR
#                 ${CMAKE_CURRENT_SOURCE_DIR}/android)
# For more information, see https://doc.qt.io/qt-6/qt-add-executable.html#target-creation
else()
    if(ANDROID)
        add_library(JuankaEmika SHARED
            ${PROJECT_SOURCES}
        )
# Define properties for Android with Qt 5 after find_package() calls as:
#    set(ANDROID_PACKAGE_SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/android")
    else()
        add_executable(JuankaEmika
            ${PROJECT_SOURCES}
        )
    endif()
endif()

target_link_libraries(JuankaEmika PRIVATE Qt${QT_VERSION_MAJOR}::Widgets
                     dqrobotics
                     robot_interface_franka)

set_target_properties(JuankaEmika PROPERTIES
    MACOSX_BUNDLE_GUI_IDENTIFIER my.example.com
    MACOSX_BUNDLE_BUNDLE_VERSION ${PROJECT_VERSION}
    MACOSX_BUNDLE_SHORT_VERSION_STRING ${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}
    MACOSX_BUNDLE TRUE
    WIN32_EXECUTABLE TRUE
)

install(TARGETS JuankaEmika
    BUNDLE DESTINATION .
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR})

if(QT_VERSION_MAJOR EQUAL 6)
    qt_finalize_executable(JuankaEmika)
endif()
