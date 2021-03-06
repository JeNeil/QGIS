# Standard includes and utils to compile into all tests.

#####################################################
# Don't forget to include output directory, otherwise
# the UI file won't be wrapped!
include_directories(${CMAKE_CURRENT_SOURCE_DIR} 
  ${CMAKE_CURRENT_BINARY_DIR}
  ${CMAKE_SOURCE_DIR}/src/core
  ${CMAKE_SOURCE_DIR}/src/core/auth
  ${CMAKE_SOURCE_DIR}/src/core/expression
  ${CMAKE_SOURCE_DIR}/src/core/geometry
  ${CMAKE_SOURCE_DIR}/src/core/labeling
  ${CMAKE_SOURCE_DIR}/src/core/mesh
  ${CMAKE_SOURCE_DIR}/src/core/metadata
  ${CMAKE_SOURCE_DIR}/src/core/network
  ${CMAKE_SOURCE_DIR}/src/core/pal
  ${CMAKE_SOURCE_DIR}/src/core/pointcloud
  ${CMAKE_SOURCE_DIR}/src/core/project
  ${CMAKE_SOURCE_DIR}/src/core/raster
  ${CMAKE_SOURCE_DIR}/src/core/symbology
  ${CMAKE_SOURCE_DIR}/src/core/effects
  ${CMAKE_SOURCE_DIR}/src/core/layertree
  ${CMAKE_SOURCE_DIR}/src/core/textrenderer
  ${CMAKE_SOURCE_DIR}/src/core/vector
  ${CMAKE_SOURCE_DIR}/src/providers/wms
  ${CMAKE_SOURCE_DIR}/src/providers/postgres
  ${CMAKE_SOURCE_DIR}/src/providers/mdal
  ${CMAKE_SOURCE_DIR}/src/providers/ogr
  ${CMAKE_SOURCE_DIR}/src/test
  ${CMAKE_SOURCE_DIR}/external
  ${CMAKE_SOURCE_DIR}/external/nlohmann

  ${CMAKE_BINARY_DIR}/src/core
)
include_directories(SYSTEM
  ${QT_INCLUDE_DIR}
  ${GEOS_INCLUDE_DIR}
  ${GDAL_INCLUDE_DIR}
  ${MDAL_INCLUDE_DIR}
  ${QCA_INCLUDE_DIR}
  ${QTKEYCHAIN_INCLUDE_DIR}
  ${POSTGRES_INCLUDE_DIR}
)

#note for tests we should not include the moc of our
#qtests in the executable file list as the moc is
#directly included in the sources
#and should not be compiled twice. Trying to include
#them in will cause an error at build time 

#No relinking and full RPATH for the install tree
#See: http://www.cmake.org/Wiki/CMake_RPATH_handling#No_relinking_and_full_RPATH_for_the_install_tree

macro (ADD_QGIS_TEST testname testsrc)
  set(qgis_${testname}_SRCS ${testsrc} ${util_SRCS})
  add_custom_target(qgis_${testname}moc ALL DEPENDS ${qgis_${testname}_MOC_SRCS})
  add_executable(qgis_${testname} ${qgis_${testname}_SRCS})
  target_link_libraries(qgis_${testname}
    ${Qt5Xml_LIBRARIES}
    ${Qt5Core_LIBRARIES}
    ${Qt5Svg_LIBRARIES}
    ${Qt5Test_LIBRARIES}
    ${PROJ_LIBRARY}
    ${GEOS_LIBRARY}
    ${GDAL_LIBRARY}
    qgis_core)
  add_test(qgis_${testname} ${CMAKE_CURRENT_BINARY_DIR}/../../../output/bin/qgis_${testname} -maxwarnings 10000)
endmacro (ADD_QGIS_TEST)

#############################################################
# Tests:

ADD_QGIS_TEST(wcsprovidertest testqgswcsprovider.cpp)
# Temporarily set to old version until server is reconfigured
#set(TEST_SERVER_URL "http://wcs.qgis.org/${COMPLETE_VERSION}")
set(TEST_SERVER_URL "http://wcs.qgis.org/1.9.0")
#add_definitions(-DTEST_SERVER_URL="${TEST_SERVER_URL}")
set_target_properties(qgis_wcsprovidertest PROPERTIES
  COMPILE_FLAGS "-DTEST_SERVER_URL=\\\"${TEST_SERVER_URL}\\\""
)

ADD_QGIS_TEST(wmscapabilitiestest
              testqgswmscapabilities.cpp)
target_link_libraries(qgis_wmscapabilitiestest wmsprovider_a qgis_core)

ADD_QGIS_TEST(wmsccapabilitiestest
              testqgswmsccapabilities.cpp)
target_link_libraries(qgis_wmsccapabilitiestest wmsprovider_a qgis_core)

ADD_QGIS_TEST(wmsprovidertest
              testqgswmsprovider.cpp)
target_link_libraries(qgis_wmsprovidertest wmsprovider_a qgis_core)

ADD_QGIS_TEST(postgresprovidertest testqgspostgresprovider.cpp)
target_link_libraries(qgis_postgresprovidertest postgresprovider_a qgis_core)

ADD_QGIS_TEST(postgresconntest testqgspostgresconn.cpp)
target_link_libraries(qgis_postgresconntest postgresprovider_a qgis_core)

if (NOT FORCE_STATIC_PROVIDERS)
  ADD_QGIS_TEST(mdalprovidertest testqgsmdalprovider.cpp)
endif()

if (WITH_EPT)
  include_directories(
    ${CMAKE_SOURCE_DIR}/src/core/providers/ept
  )
  ADD_QGIS_TEST(eptprovidertest testqgseptprovider.cpp)
endif()

if (WITH_PDAL)
  include_directories(
    ${CMAKE_SOURCE_DIR}/src/providers/pdal
  )
  ADD_QGIS_TEST(pdalprovidertest testqgspdalprovider.cpp)
  target_link_libraries(qgis_pdalprovidertest pdalprovider_a qgis_core)
endif()

#############################################################
# WCS public servers test:
# No need to test on all platforms
if(UNIX AND NOT ANDROID AND CMAKE_BUILD_TYPE MATCHES Debug)
  set ( WCSTEST_SRCS
        ../../../src/providers/wcs/qgswcscapabilities.cpp
        testqgswcspublicservers.cpp
  )

  add_executable ( qgis_wcstest ${WCSTEST_SRCS} )

  include_directories(${CMAKE_CURRENT_SOURCE_DIR}
    ${CMAKE_CURRENT_SOURCE_DIR}/../../../src/core
    ${CMAKE_CURRENT_SOURCE_DIR}/../../../src/core/raster
    ${CMAKE_CURRENT_SOURCE_DIR}/../../../src/providers/wcs
  )

  target_link_libraries(qgis_wcstest
    ${Qt5Core_LIBRARIES}
    ${Qt5Network_LIBRARIES}
    ${Qt5Svg_LIBRARIES}
    ${Qt5Xml_LIBRARIES}
    ${OPTIONAL_QTWEBKIT}
    qgis_core
  )

  install (TARGETS qgis_wcstest
    BUNDLE DESTINATION ${QGIS_BIN_DIR}
    RUNTIME DESTINATION ${QGIS_BIN_DIR}
  )
  install(FILES wcs-servers.json DESTINATION ${QGIS_DATA_DIR}/resources)

endif()

add_subdirectory(grass)
