cmake_minimum_required(VERSION 3.0)
project(rapidcheck CXX)

# Don't warn about symbol visibility for static libraries with CMake 3.3 and later.
if(POLICY CMP0063)
  cmake_policy(SET CMP0063 NEW)
endif()

enable_testing()

option(RC_ENABLE_TESTS "Build RapidCheck tests" OFF)
option(RC_ENABLE_EXAMPLES "Build RapidCheck examples" OFF)

if(MSVC)
  # /bigobj - some object files become very large so we need this
  # /wd4503 - truncation of decorated name, not much we can do about it so
  #           disable it
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /bigobj /wd4503 /WX")
  # /RTC* is incompatible with /O2 needed for Random.cpp to speed it up
  string(REGEX REPLACE "/RTC(su|[1su])" ""
    CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG}")
else()
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Werror -Wno-missing-braces -std=c++11")
  if(APPLE)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -stdlib=libc++")
  endif()
endif()

add_library(rapidcheck
  src/BeforeMinimalTestCase.cpp
  src/Check.cpp
  src/Classify.cpp
  src/GenerationFailure.cpp
  src/Log.cpp
  src/Random.cpp
  src/Show.cpp
  src/detail/Any.cpp
  src/detail/Assertions.cpp
  src/detail/Base64.cpp
  src/detail/Configuration.cpp
  src/detail/DefaultTestListener.cpp
  src/detail/FrequencyMap.cpp
  src/detail/ImplicitParam.cpp
  src/detail/LogTestListener.cpp
  src/detail/MapParser.cpp
  src/detail/MulticastTestListener.cpp
  src/detail/ParseException.cpp
  src/detail/Platform.cpp
  src/detail/Property.cpp
  src/detail/PropertyContext.cpp
  src/detail/ReproduceListener.cpp
  src/detail/Results.cpp
  src/detail/Serialization.cpp
  src/detail/StringSerialization.cpp
  src/detail/TestMetadata.cpp
  src/detail/TestParams.cpp
  src/detail/Testing.cpp
  src/gen/Numeric.cpp
  src/gen/Text.cpp
  src/gen/detail/ExecHandler.cpp
  src/gen/detail/GenerationHandler.cpp
  src/gen/detail/Recipe.cpp
  src/gen/detail/ScaleInteger.cpp
  )

# Random is used a LOT so it should preferrably be really fast.
if(MSVC)
  set_property(SOURCE src/Random.cpp
    APPEND_STRING PROPERTY COMPILE_FLAGS " /O2")
else()
  set_property(SOURCE src/Random.cpp
    APPEND_STRING PROPERTY COMPILE_FLAGS " -O3")
endif()



target_include_directories(rapidcheck PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include/>
    $<INSTALL_INTERFACE:include/rapidcheck>  # <prefix>/include/rapidcheck
)

include(GNUInstallDirs)
install(TARGETS rapidcheck EXPORT rapidcheckConfig
    ARCHIVE  DESTINATION ${CMAKE_INSTALL_LIBDIR}
    LIBRARY  DESTINATION ${CMAKE_INSTALL_LIBDIR}
    RUNTIME  DESTINATION ${CMAKE_INSTALL_BINDIR})  # This is for Windows
install(DIRECTORY include/ DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

install(EXPORT rapidcheckConfig DESTINATION share/rapidcheck/cmake)

export(TARGETS rapidcheck FILE rapidcheckConfig.cmake)



# On Windows under MinGW, random_device provides no entropy,
# so it will always return the same value.
# Seed using system time instead.
# See: https://stackoverflow.com/questions/18880654/why-do-i-get-the-same-sequence-for-every-run-with-stdrandom-device-with-mingw
if(MINGW)
  target_compile_definitions(rapidcheck PRIVATE RC_SEED_SYSTEM_TIME)
endif()

add_subdirectory(ext)

if (RC_ENABLE_TESTS)
  add_subdirectory(test)
endif()

if (RC_ENABLE_EXAMPLES)
  add_subdirectory(examples)
endif()

add_subdirectory(extras)
