########################
####### Runtime-neutral
########################
set(ttg-util-header
        ${CMAKE_CURRENT_SOURCE_DIR}/ttg/util/backtrace.h
        ${CMAKE_CURRENT_SOURCE_DIR}/ttg/util/bug.h
        ${CMAKE_CURRENT_SOURCE_DIR}/ttg/util/demangle.h
        ${CMAKE_CURRENT_SOURCE_DIR}/ttg/util/dot.h
        ${CMAKE_CURRENT_SOURCE_DIR}/ttg/util/future.h
        ${CMAKE_CURRENT_SOURCE_DIR}/ttg/util/hash.h
        ${CMAKE_CURRENT_SOURCE_DIR}/ttg/util/macro.h
        ${CMAKE_CURRENT_SOURCE_DIR}/ttg/util/meta.h
        ${CMAKE_CURRENT_SOURCE_DIR}/ttg/util/print.h
        ${CMAKE_CURRENT_SOURCE_DIR}/ttg/util/trace.h
        ${CMAKE_CURRENT_SOURCE_DIR}/ttg/util/tree.h
        ${CMAKE_CURRENT_SOURCE_DIR}/ttg/util/version.h
        ${CMAKE_CURRENT_SOURCE_DIR}/ttg/util/void.h
    )
set(ttg-base-header
        ${CMAKE_CURRENT_SOURCE_DIR}/ttg/base/keymap.h
        ${CMAKE_CURRENT_SOURCE_DIR}/ttg/base/op.h
        ${CMAKE_CURRENT_SOURCE_DIR}/ttg/base/terminal.h
        ${CMAKE_CURRENT_SOURCE_DIR}/ttg/base/world.h
    )
set(ttg-impl-header
        ${CMAKE_CURRENT_SOURCE_DIR}/ttg/broadcast.h
        ${CMAKE_CURRENT_SOURCE_DIR}/ttg/edge.h
        ${CMAKE_CURRENT_SOURCE_DIR}/ttg/execution.h
        ${CMAKE_CURRENT_SOURCE_DIR}/ttg/func.h
        ${CMAKE_CURRENT_SOURCE_DIR}/ttg/fwd.h
        ${CMAKE_CURRENT_SOURCE_DIR}/ttg/impl_selector.h
        ${CMAKE_CURRENT_SOURCE_DIR}/ttg/op.h
        ${CMAKE_CURRENT_SOURCE_DIR}/ttg/reduce.h
        ${CMAKE_CURRENT_SOURCE_DIR}/ttg/runtimes.h
        ${CMAKE_CURRENT_SOURCE_DIR}/ttg/terminal.h
        ${CMAKE_CURRENT_SOURCE_DIR}/ttg/traverse.h
        ${CMAKE_CURRENT_SOURCE_DIR}/ttg/world.h
        ${CMAKE_CURRENT_SOURCE_DIR}/ttg/wrap.h
    )
set(ttg-header
        ${CMAKE_CURRENT_SOURCE_DIR}/ttg.h
    )
set(ttg-sources
        ${ttg-header}
        ${ttg-impl-header}
        ${ttg-base-header}
        ${ttg-util-header}
        ${CMAKE_CURRENT_SOURCE_DIR}/ttg/util/backtrace.cpp
        ${CMAKE_CURRENT_SOURCE_DIR}/ttg/util/bug.cpp
        ${CMAKE_CURRENT_SOURCE_DIR}/ttg/util/version.cc
    )
add_library(ttg ${ttg-sources})
target_include_directories(ttg PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
        $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
        )
target_link_libraries(ttg PUBLIC Boost::boost)
# optional dependencies
if (TARGET TTG_Libunwind)
  target_link_libraries(ttg PUBLIC TTG_Libunwind)
endif(TARGET TTG_Libunwind)

install(FILES ${ttg-header}
        DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
        COMPONENT ttg)
install(FILES ${ttg-impl-header}
        DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/ttg/
        COMPONENT ttg)
install(FILES ${ttg-util-header}
        DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/ttg/util
        COMPONENT ttg)
install(FILES ${ttg-base-header}
        DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/ttg/base
        COMPONENT ttg)

########################
####### Serialization
########################
set(ttg-serialization-source
        ${CMAKE_CURRENT_SOURCE_DIR}/ttg/serialization.h
        ${CMAKE_CURRENT_SOURCE_DIR}/ttg/serialization/backends.h
        ${CMAKE_CURRENT_SOURCE_DIR}/ttg/serialization/buffer_archive.cpp
        ${CMAKE_CURRENT_SOURCE_DIR}/ttg/serialization/buffer_archive.h
        ${CMAKE_CURRENT_SOURCE_DIR}/ttg/serialization/data_descriptor.h
        ${CMAKE_CURRENT_SOURCE_DIR}/ttg/serialization/stream.h
        ${CMAKE_CURRENT_SOURCE_DIR}/ttg/serialization/traits.h
        ${CMAKE_CURRENT_SOURCE_DIR}/ttg/serialization/backends/boost.h
        ${CMAKE_CURRENT_SOURCE_DIR}/ttg/serialization/backends/boost/archive.h
        ${CMAKE_CURRENT_SOURCE_DIR}/ttg/serialization/backends/cereal.h
        ${CMAKE_CURRENT_SOURCE_DIR}/ttg/serialization/backends/madness.h
        ${CMAKE_CURRENT_SOURCE_DIR}/ttg/serialization/std/allocator.h
        ${CMAKE_CURRENT_SOURCE_DIR}/ttg/serialization/std/array.h
        ${CMAKE_CURRENT_SOURCE_DIR}/ttg/serialization/std/list.h
        ${CMAKE_CURRENT_SOURCE_DIR}/ttg/serialization/std/pair.h
        ${CMAKE_CURRENT_SOURCE_DIR}/ttg/serialization/std/tuple.h
        ${CMAKE_CURRENT_SOURCE_DIR}/ttg/serialization/std/vector.h
   )
add_library(ttg-serialization
        ${ttg-serialization-source})
if (TARGET MADworld)
  target_link_libraries(ttg-serialization PUBLIC MADworld)
  target_compile_definitions(ttg-serialization PUBLIC TTG_SERIALIZATION_SUPPORTS_MADNESS=1)
  # make madness-only serialization target
  add_library(ttg-serialization-madness
          ${ttg-serialization-source})
  target_link_libraries(ttg-serialization-madness PUBLIC MADworld)
  target_compile_definitions(ttg-serialization-madness PUBLIC TTG_SERIALIZATION_SUPPORTS_MADNESS=1)
endif(TARGET MADworld)
if (TARGET Boost::serialization)
  target_link_libraries(ttg-serialization PUBLIC Boost::serialization)
  target_compile_definitions(ttg-serialization PUBLIC TTG_SERIALIZATION_SUPPORTS_BOOST=1)
  # make boost-only serialization target
  add_library(ttg-serialization-boost
          ${ttg-serialization-source})
  target_link_libraries(ttg-serialization-boost PUBLIC Boost::serialization)
  target_compile_definitions(ttg-serialization-boost PUBLIC TTG_SERIALIZATION_SUPPORTS_BOOST=1)
endif(TARGET Boost::serialization)
if (TARGET cereal::cereal)
  target_link_libraries(ttg-serialization PUBLIC cereal::cereal)
  target_compile_definitions(ttg-serialization PUBLIC TTG_SERIALIZATION_SUPPORTS_CEREAL=1)
  # make cereal-only serialization target
  add_library(ttg-serialization-cereal
          ${ttg-serialization-source})
  target_link_libraries(ttg-serialization-cereal PUBLIC cereal::cereal)
  target_compile_definitions(ttg-serialization-cereal PUBLIC TTG_SERIALIZATION_SUPPORTS_CEREAL=1)
endif(TARGET cereal::cereal)

foreach(target ttg-serialization;ttg-serialization-madness;ttg-serialization-boost;ttg-serialization-cereal)
  if (TARGET ${target})
    target_include_directories(${target} PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
        $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
        )
  endif(TARGET ${target})
endforeach()

#########################
####### MADNESS-specific
#########################
if (TARGET MADworld)
  add_library(ttg-mad INTERFACE)
  target_link_libraries(ttg-mad INTERFACE ttg Boost::boost)
  target_include_directories(ttg-mad INTERFACE
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
    $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
  )
  set(ttg-mad_headers
          ttg/madness/fwd.h
          ttg/madness/import.h
          ttg/madness/ttg.h
          ttg/madness/watch.h)
  target_sources(ttg-mad INTERFACE ${ttg-mad_headers})
  install(FILES ${ttg-mad_headers}
          DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/ttg/madness/
          COMPONENT ttg-mad)
  # Use current CMAKE_CXX_FLAGS to compile targets dependent on this library
  string (REPLACE " " ";" CMAKE_CXX_FLAG_LIST "${CMAKE_CXX_FLAGS}")
  target_compile_options(ttg-mad INTERFACE $<INSTALL_INTERFACE:${CMAKE_CXX_FLAG_LIST}>)
  target_link_libraries(ttg-mad INTERFACE MADworld)
  # ttg-mad can use MADNESS serialization only
  target_link_libraries(ttg-mad INTERFACE ttg-serialization-madness)
endif(TARGET MADworld)

########################
####### PaRSEC-specific
########################
if (TARGET PaRSEC::parsec)
  add_library(ttg-parsec INTERFACE)
  find_package(MPI)
  target_link_libraries(ttg-parsec INTERFACE ttg Boost::boost MPI::MPI_CXX)
  target_include_directories(ttg-parsec INTERFACE
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
    $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
  )
  set(ttg-parsec_headers
          ttg/parsec/fwd.h
          ttg/parsec/import.h
          ttg/parsec/ttg.h)
  target_sources(ttg-parsec INTERFACE ${ttg-parsec_headers})
  install(FILES ${ttg-parsec_headers}
          DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/ttg/parsec/
          COMPONENT ttg-parsec)
  # Use current CMAKE_CXX_FLAGS to compile targets dependent on this library
  string (REPLACE " " ";" CMAKE_CXX_FLAG_LIST "${CMAKE_CXX_FLAGS}")
  target_compile_options(ttg-parsec INTERFACE $<INSTALL_INTERFACE:${CMAKE_CXX_FLAG_LIST}>)
  if (NOT TARGET MADworld)
    message(FATAL_ERROR "PaRSEC backend requires MADworld for serialization (this requirement is to be lifted soon)")
  endif(NOT TARGET MADworld)
  target_link_libraries(ttg-parsec INTERFACE PaRSEC::parsec)
  # parsec depends on TTG's serialization layer since it does not provide its own
  if (TTG_PARSEC_USE_BOOST_SERIALIZATION AND TARGET ttg-serialization-boost)
    target_link_libraries(ttg-parsec INTERFACE ttg-serialization-boost)
  else()
    target_link_libraries(ttg-parsec INTERFACE ttg-serialization)
  endif()
endif(TARGET PaRSEC::parsec)
