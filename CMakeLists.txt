cmake_minimum_required(VERSION 3.31)
set(CMAKE_PROJECT_TOP_LEVEL_INCLUDES .devcontainer/cmake/xproinc.cmake)
project(openssl LANGUAGES C ASM)
include(GNUInstallDirs)
include(xpflags)
########################################
set_property(GLOBAL PROPERTY USE_FOLDERS ON) # enables MSVC Solution Folders
# NOTE: attempted to make OPENSSL_DEFS match other build systems (node.js and openssl perl)
set(OPENSSL_DEFS
  -D_REENTRANT
  -DL_ENDIAN
  -DOPENSSL_PIC
  -DOPENSSLDIR=\"/etc/ssl\"
  -DENGINESDIR=\"/dev/null\"
  )
if(WIN32)
  list(APPEND OPENSSL_DEFS
    -DOPENSSL_SYS_WIN32
    -DWIN32_LEAN_AND_MEAN
    -DUNICODE -D_UNICODE
    -D_CRT_SECURE_NO_DEPRECATE
    -D_WINSOCK_DEPRECATED_NO_WARNINGS
    )
  set(asmDir asmfromnode/archs/VC-WIN64A/asm)
elseif(UNIX)
  list(APPEND OPENSSL_DEFS
    -DOPENSSL_USE_NODELETE
    )
  set(asmDir asmfromnode/archs/linux-${CMAKE_SYSTEM_PROCESSOR}/asm)
endif()
if(CMAKE_SYSTEM_PROCESSOR MATCHES "x86_64|AMD64")
  if(WIN32)
    xpFindPkg(PKGS nasm)
    xpGetPkgVar(nasm EXE) # sets NASM_EXE
    set(CMAKE_ASM_NASM_COMPILER ${NASM_EXE})
    enable_language(ASM_NASM)
  endif()
endif()
if(CMAKE_ASM_NASM_COMPILER_LOADED AND EXISTS ${CMAKE_SOURCE_DIR}/${asmDir} AND IS_DIRECTORY ${CMAKE_SOURCE_DIR}/${asmDir})
  set(ASM_BUILD TRUE)
  list(APPEND OPENSSL_DEFS
    -DOPENSSL_CPUID_OBJ
    -DOPENSSL_IA32_SSE2
    -DOPENSSL_BN_ASM_MONT
    -DOPENSSL_BN_ASM_GF2m
    -DSHA1_ASM # must define L_ENDIAN for x86
    -DSHA256_ASM
    -DSHA512_ASM
    -DRC4_ASM
    -DMD5_ASM
    -DVPAES_ASM
    -DGHASH_ASM
    -DECP_NISTZ256_ASM
    -DPOLY1305_ASM
    )
  if(WIN32)
    list(APPEND OPENSSL_DEFS
      -DRMD160_ASM
      -DAES_ASM
      -DWHIRLPOOL_ASM
      -DPADLOCK_ASM
      )
  elseif(UNIX)
    list(APPEND OPENSSL_DEFS
      -DOPENSSL_BN_ASM_MONT5
      -DKECCAK1600_ASM
      -DX25519_ASM
      )
  endif()
  message(STATUS "ASM_BUILD: ${ASM_BUILD}")
else()
  set(ASM_BUILD FALSE)
  list(APPEND OPENSSL_DEFS
    -DOPENSSL_NO_ASM
    )
  message(STATUS "ASM_BUILD: ${ASM_BUILD}")
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
set(libs crypto ssl)
if(DEFINED XP_NAMESPACE)
  set(nameSpace NAMESPACE ${XP_NAMESPACE}::)
  list(TRANSFORM libs PREPEND "${XP_NAMESPACE}::")
endif()
if(NOT DEFINED XP_INSTALL_CMAKEDIR)
  set(XP_INSTALL_CMAKEDIR ${CMAKE_INSTALL_DATADIR}/cmake)
endif()
set(targetsFile ${PROJECT_NAME}-targets)
string(JOIN "\n" EXT1
  "set(THREAD_PREFER_PTHREAD_FLAG ON)"
  "find_package(Threads REQUIRED) # crypto depends on Threads::Threads"
  ""
  )
xpPackageDevel(TARGETS_FILE ${targetsFile} LIBRARIES ${libs})
########################################
add_subdirectory(include)
add_subdirectory(crypto)
add_subdirectory(ssl)
add_subdirectory(apps)
add_subdirectory(test)
install(EXPORT ${targetsFile} DESTINATION ${XP_INSTALL_CMAKEDIR} ${nameSpace})
