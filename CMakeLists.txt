cmake_minimum_required(VERSION 3.31)
set(CMAKE_PROJECT_TOP_LEVEL_INCLUDES .devcontainer/cmake/xproinc.cmake)
project(openssl LANGUAGES C ASM)
set_property(GLOBAL PROPERTY USE_FOLDERS ON) # enables MSVC Solution Folders
########################################
# NOTE: attempted to make OPENSSL_DEFS match other build systems (node.js and openssl perl)
set(OPENSSL_DEFS
  -D_REENTRANT
  -DL_ENDIAN # NOTE: SHA1_ASM must define L_ENDIAN for x86
  -DOPENSSL_PIC
  [[-DOPENSSLDIR="/etc/ssl"]]
  [[-DENGINESDIR="/dev/null"]]
  )
if(WIN32)
  list(APPEND OPENSSL_DEFS
    -DOPENSSL_SYS_WIN32
    -DWIN32_LEAN_AND_MEAN
    -DUNICODE -D_UNICODE
    -D_CRT_SECURE_NO_DEPRECATE
    -D_WINSOCK_DEPRECATED_NO_WARNINGS
    )
elseif(UNIX)
  list(APPEND OPENSSL_DEFS
    -DOPENSSL_USE_NODELETE
    )
endif()
########################################
include(FetchContent)
FetchContent_Declare(node
  GIT_REPOSITORY https://github.com/nodejs/node
  GIT_TAG v14.17.6 # for openssl 1.1.1l
  GIT_SHALLOW TRUE
  )
FetchContent_MakeAvailable(node)
# assembly from node.js
include(asmfromnode/copyfromnode.cmake)
set(ASM_BUILD FALSE)
if(CMAKE_SYSTEM_PROCESSOR MATCHES "x86_64|AMD64")
  if(WIN32)
    set(asmDir ${CMAKE_SOURCE_DIR}/asmfromnode/archs/VC-WIN64A/asm)
    xpFindPkg(PKGS nasm)
    xpGetPkgVar(nasm EXE) # sets NASM_EXE
    set(CMAKE_ASM_NASM_COMPILER ${NASM_EXE})
    enable_language(ASM_NASM)
  elseif(UNIX)
    set(asmDir ${CMAKE_SOURCE_DIR}/asmfromnode/archs/linux-${CMAKE_SYSTEM_PROCESSOR}/asm)
    xpFindPkg(PKGS yasm)
    xpGetPkgVar(yasm EXE) # sets YASM_EXE
    get_target_property(yasmExe ${YASM_EXE} LOCATION)
    set(CMAKE_ASM_NASM_COMPILER ${yasmExe})
    enable_language(ASM_NASM)
  endif()
  if(CMAKE_ASM_NASM_COMPILER_LOADED AND EXISTS ${asmDir} AND IS_DIRECTORY ${asmDir})
    set(ASM_BUILD TRUE)
  endif()
elseif(CMAKE_SYSTEM_PROCESSOR MATCHES "arm64|aarch64")
  enable_language(ASM)
  if(APPLE)
    set(asmDir ${CMAKE_SOURCE_DIR}/asmfromnode/archs/darwin64-arm64-cc/asm)
  elseif(UNIX)
    set(asmDir ${CMAKE_SOURCE_DIR}/asmfromnode/archs/linux-aarch64/asm)
  endif()
  if(CMAKE_ASM_COMPILER_LOADED AND EXISTS ${asmDir} AND IS_DIRECTORY ${asmDir})
    set(ASM_BUILD TRUE)
  endif()
endif()
if(ASM_BUILD)
  message(STATUS "ASM directory: ${asmDir}")
else()
  list(APPEND OPENSSL_DEFS
    -DOPENSSL_NO_ASM
    )
endif()
add_definitions(${OPENSSL_DEFS})
########################################
function(compareCmakeFilesys files)
  file(GLOB_RECURSE filesys RELATIVE ${CMAKE_CURRENT_LIST_DIR} *)
  file(GLOB_RECURSE ignorefiles RELATIVE ${CMAKE_CURRENT_LIST_DIR} .*.swp)
  if(ignorefiles)
    list(REMOVE_ITEM filesys ${ignorefiles})
  endif()
  foreach(f ${filesys})
    list(FIND files ${f} index)
    if(${index} GREATER -1)
      list(REMOVE_AT files ${index})
      list(REMOVE_ITEM filesys ${f})
    endif()
  endforeach()
  if(filesys)
    message(FATAL_ERROR "files not in cmake: ${filesys}")
  endif()
  foreach(f ${files})
    get_filename_component(absPath ${f} ABSOLUTE)
    if(NOT absPath MATCHES CMAKE_CURRENT_LIST_DIR)
      # remove any files that aren't under CMAKE_CURRENT_LIST_DIR
      list(REMOVE_ITEM files ${f})
    endif()
  endforeach()
  if(files)
    message(FATAL_ERROR "files not in repo, but in cmake: ${files}")
  endif()
endfunction()
########################################
set(targetsFile ${PROJECT_NAME}-targets)
if(DEFINED XP_NAMESPACE)
  string(JOIN "\n" ALIASES
    "if(NOT TARGET OpenSSL::Crypto)"
    "  add_library(OpenSSL::Crypto ALIAS ${XP_NAMESPACE}::crypto)"
    "endif()"
    ""
    "if(NOT TARGET OpenSSL::SSL)"
    "  add_library(OpenSSL::SSL ALIAS ${XP_NAMESPACE}::ssl)"
    "endif()"
    ""
    )
  xpExternPackage(NAMESPACE ${XP_NAMESPACE}
    TARGETS_FILE ${targetsFile} LIBRARIES crypto ssl
    BASE OpenSSL_1_1_1l XPDIFF "intro"
    FIND_THREADS PVT_DEPS nasm yasm
    WEB "http://www.openssl.org/" UPSTREAM "github.com/openssl/openssl"
    DESC "Cryptography and SSL/TLS Toolkit"
    LICENSE "[OpenSSL AND SSLeay](http://www.openssl.org/source/license.html 'dual OpenSSL and SSLeay License: both apply')"
    )
  set(nameSpace NAMESPACE ${XP_NAMESPACE}::)
endif()
########################################
add_subdirectory(include)
add_subdirectory(crypto)
add_subdirectory(ssl)
add_subdirectory(apps)
add_subdirectory(test)
install(EXPORT ${targetsFile} DESTINATION ${CMAKE_INSTALL_CMAKEDIR} ${nameSpace})
