#
# Copyright (c) 2008-2016 the Urho3D project.
#
# Permission is hereby granted, free of charge, to any person obtaining a copy
# of this software and associated documentation files (the "Software"), to deal
# in the Software without restriction, including without limitation the rights
# to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
# copies of the Software, and to permit persons to whom the Software is
# furnished to do so, subject to the following conditions:
#
# The above copyright notice and this permission notice shall be included in
# all copies or substantial portions of the Software.
#
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
# IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
# AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
# OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
# THE SOFTWARE.
#

cmake_minimum_required(VERSION 3.0)

project(AngelScript VERSION 1.0.0)

set(SOURCE_FILES
  source/as_atomic.cpp
  source/as_builder.cpp
  source/as_bytecode.cpp
  source/as_callfunc.cpp
  source/as_callfunc_arm.cpp
  source/as_callfunc_mips.cpp
  source/as_callfunc_ppc.cpp
  source/as_callfunc_ppc_64.cpp
  source/as_callfunc_sh4.cpp
  source/as_callfunc_x64_gcc.cpp
  source/as_callfunc_x64_mingw.cpp
  source/as_callfunc_x64_msvc.cpp
  source/as_callfunc_x86.cpp
  source/as_callfunc_xenon.cpp
  source/as_compiler.cpp
  source/as_configgroup.cpp
  source/as_context.cpp
  source/as_datatype.cpp
  source/as_gc.cpp
  source/as_generic.cpp
  source/as_globalproperty.cpp
  source/as_memory.cpp
  source/as_module.cpp
  source/as_objecttype.cpp
  source/as_outputbuffer.cpp
  source/as_parser.cpp
  source/as_restore.cpp
  source/as_scriptcode.cpp
  source/as_scriptengine.cpp
  source/as_scriptfunction.cpp
  source/as_scriptnode.cpp
  source/as_scriptobject.cpp
  source/as_string.cpp
  source/as_string_util.cpp
  source/as_thread.cpp
  source/as_tokenizer.cpp
  source/as_typeinfo.cpp
  source/as_variablescope.cpp
  )

# Define source files
if (IOS OR ANDROID OR RPI)
    enable_language (ASM)
    if (IOS)
        set (ASM_FILES source/as_callfunc_arm_xcode.S)
    else ()
        set (ASM_FILES source/as_callfunc_arm_gcc.S)
    endif ()
elseif (MSVC AND URHO3D_64BIT)
    enable_language (ASM_MASM)
    set(ASM_FILES source/as_callfunc_x64_msvc_asm.asm)
endif ()
list (APPEND SOURCE_FILES ${ASM_FILES})

add_library(AngelScript "${SOURCE_FILES}")

find_package(Threads REQUIRED)
target_link_libraries(AngelScript PUBLIC Threads::Threads)

#### install

set(TARGET_PUBLIC_HDRS include/angelscript.h)

set(config_install_dir "lib/cmake/${PROJECT_NAME}")
set(include_install_dir "include")

set(generated_dir "${CMAKE_CURRENT_BINARY_DIR}/generated")

# Configuration
set(version_config "${generated_dir}/${PROJECT_NAME}ConfigVersion.cmake")
set(project_config "${generated_dir}/${PROJECT_NAME}Config.cmake")
set(targets_export_name "${PROJECT_NAME}Targets")
set(namespace "${PROJECT_NAME}::")

include(CMakePackageConfigHelpers)

write_basic_package_version_file(
  "${version_config}" COMPATIBILITY SameMajorVersion
  )

configure_package_config_file(
  "cmake/Config.cmake.in"
  "${project_config}"
  INSTALL_DESTINATION "${config_install_dir}"
  )

install(
  TARGETS AngelScript
  EXPORT "${targets_export_name}"
  LIBRARY DESTINATION "lib"
  ARCHIVE DESTINATION "lib"
  RUNTIME DESTINATION "bin"
  INCLUDES DESTINATION "${include_install_dir}"
  )

foreach(HEADER ${TARGET_PUBLIC_HDRS})
  install(
    FILES "${CMAKE_CURRENT_SOURCE_DIR}/${HEADER}"
    DESTINATION "${include_install_dir}/${PROJECT_NAME}"
    )
endforeach()

install(
  DIRECTORY AngelScript # No trailing slash
  DESTINATION "${include_install_dir}/"
  FILES_MATCHING PATTERN "*.h" PATTERN "*.hpp"
  )

install(
  FILES "${project_config}" "${version_config}"
  DESTINATION "${config_install_dir}"
  )

install(
  EXPORT "${targets_export_name}"
  NAMESPACE "${namespace}"
  DESTINATION "${config_install_dir}"
  )
