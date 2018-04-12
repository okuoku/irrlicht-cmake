project(irr-helloworld)
cmake_minimum_required(VERSION 2.8.11)

# Configure the engine
add_definitions(
    -D_IRR_STATIC_LIB_ # Mandatory (uncool)
    -DNO_IRR_COMPILE_WITH_DIRECT3D_9_
    -DNO_IRR_COMPILE_WITH_WEBGL1_ # FIXME: Wrong eval. order..?
    -DNO_IRR_COMPILE_WITH_OPENGL_
    -DNO_IRR_COMPILE_WITH_OGLES2_
    -DNO_IRR_COMPILE_WITH_OGLES1_
    -DNO_IRR_COMPILE_WITH_X11_DEVICE_)

add_subdirectory(../../ irrlicht) # => irrlicht_static

include_directories(${IRRLICHT_ROOT}/include)

add_executable(hello main.cpp)
target_link_libraries(hello irrlicht_static)
