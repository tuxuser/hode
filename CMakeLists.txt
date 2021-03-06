project(hode)

cmake_minimum_required(VERSION 3.0)

include(FindPkgConfig)

set(CMAKE_CXX_FLAGS "-g -Wall -pedantic -MMD ${CMAKE_CXX_FLAGS}")
if(VITA)
  set(ENV{PKG_CONFIG_PATH} "$ENV{VITASDK}/arm-vita-eabi/lib/pkgconfig")
endif(VITA)

pkg_check_modules(SDL2 sdl2 REQUIRED)

include_directories(
  ${SDL2_INCLUDE_DIRS}
)

file(GLOB SRC *.cpp 3p/inih/ini.c 3p/libxbr-standalone/xbr.c)
list(FILTER SRC EXCLUDE REGEX ".*android.cpp")
add_executable(${CMAKE_PROJECT_NAME}
  ${SRC}
)

target_link_libraries(${CMAKE_PROJECT_NAME}
  ${SDL2_STATIC_LIBRARIES}
)

if(SWITCH_LIBNX)
  set(CMAKE_C_FLAGS "-O2 -ffunction-sections -fdata-sections -march=armv8-a -mtune=cortex-a57 -mtp=soft -fPIE -ftls-model=local-exec")
  set(CMAKE_CXX_FLAGS "${CMAKE_C_FLAGS}")
  include_directories($ENV{DEVKITPRO}/libnx/include)
  add_definitions(-D__SWITCH__)
  add_custom_target(${CMAKE_PROJECT_NAME}.nro
    DEPENDS ${CMAKE_PROJECT_NAME}
    COMMAND nacptool --create "Heart of Darkness" "cyx, usineur" "0.2.8" ${CMAKE_PROJECT_NAME}.nacp
    COMMAND elf2nro ${CMAKE_PROJECT_NAME} ${CMAKE_PROJECT_NAME}.nro --icon=${CMAKE_SOURCE_DIR}/3p/res/icon.jpg --nacp=${CMAKE_PROJECT_NAME}.nacp
  )
  add_custom_target(nxlink
    COMMAND nxlink -a $(SWITCHIP) ${CMAKE_PROJECT_NAME}.nro -s -p ${CMAKE_PROJECT_NAME}/${CMAKE_PROJECT_NAME}.nro
    DEPENDS ${CMAKE_PROJECT_NAME}.nro
  )
endif(SWITCH_LIBNX)

if(VITA)
  include("${VITASDK}/share/vita.cmake" REQUIRED)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -O2 -fpermissive")
  vita_create_self(eboot.bin ${CMAKE_PROJECT_NAME} UNSAFE)
  vita_create_vpk(${CMAKE_PROJECT_NAME}.vpk "HODEVITA1" eboot.bin
    NAME ${CMAKE_PROJECT_NAME}
    FILE 3p/res/icon0.png sce_sys/icon0.png
    FILE 3p/res/bg.png sce_sys/livearea/contents/bg.png
    FILE 3p/res/startup.png sce_sys/livearea/contents/startup.png
    FILE 3p/res/template.xml sce_sys/livearea/contents/template.xml
)
endif(VITA)
