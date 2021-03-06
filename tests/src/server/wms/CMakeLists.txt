#####################################################
# Don't forget to include output directory, otherwise
# the UI file won't be wrapped!
include_directories(${CMAKE_CURRENT_SOURCE_DIR}
  ${CMAKE_SOURCE_DIR}/src/test
  ${CMAKE_SOURCE_DIR}/src/server
  ${CMAKE_SOURCE_DIR}/src/server/services
  ${CMAKE_SOURCE_DIR}/src/server/services/wms

  ${CMAKE_BINARY_DIR}/src/server

  ${CMAKE_CURRENT_BINARY_DIR}
)

#note for tests we should not include the moc of our
#qtests in the executable file list as the moc is
#directly included in the sources
#and should not be compiled twice. Trying to include
#them in will cause an error at build time

#No relinking and full RPATH for the install tree
#See: http://www.cmake.org/Wiki/CMake_RPATH_handling#No_relinking_and_full_RPATH_for_the_install_tree
set(MODULE_WMS_SRCS
  ${CMAKE_SOURCE_DIR}/src/server/services/wms/qgswmsrequest.cpp
  ${CMAKE_SOURCE_DIR}/src/server/services/wms/qgswmsrenderer.cpp
  ${CMAKE_SOURCE_DIR}/src/server/services/wms/qgswmsrestorer.cpp
  ${CMAKE_SOURCE_DIR}/src/server/services/wms/qgsmaprendererjobproxy.cpp
  ${CMAKE_SOURCE_DIR}/src/server/services/wms/qgswmsparameters.cpp
  ${CMAKE_SOURCE_DIR}/src/server/services/wms/qgswmsrendercontext.cpp
)

set(MODULE_WMS_HDRS
  ${CMAKE_SOURCE_DIR}/src/server/services/wms/qgswmsserviceexception.h
)

QT5_WRAP_CPP(MODULE_WMS_MOC_SRCS ${MODULE_WMS_HDRS})

macro (ADD_QGIS_TEST TESTSRC)
  set (TESTNAME  ${TESTSRC})
  string(REPLACE "test" "" TESTNAME ${TESTNAME})
  string(REPLACE "qgs" "" TESTNAME ${TESTNAME})
  string(REPLACE ".cpp" "" TESTNAME ${TESTNAME})
  set (TESTNAME  "qgis_${TESTNAME}test")
  add_executable(${TESTNAME} ${TESTSRC} ${MODULE_WMS_SRCS} ${MODULE_WMS_MOC_SRCS})
  target_compile_features(${TESTNAME} PRIVATE cxx_std_17)
  target_link_libraries(${TESTNAME}
    ${Qt5Core_LIBRARIES}
    ${Qt5Xml_LIBRARIES}
    ${Qt5Svg_LIBRARIES}
    ${Qt5Test_LIBRARIES}
    ${PROJ_LIBRARY}
    ${GEOS_LIBRARY}
    ${GDAL_LIBRARY}
    qgis_core
    qgis_server
  )
  add_test(${TESTNAME} ${CMAKE_BINARY_DIR}/output/bin/${TESTNAME} -maxwarnings 10000)
endmacro (ADD_QGIS_TEST)

#############################################################
# Tests:

set(TESTS
  test_qgsserver_wms_dxf.cpp
  test_qgsserver_wms_request.cpp
  test_qgsserver_wms_restorer.cpp
  test_qgsserver_wms_exceptions.cpp
  test_qgsserver_wms_parameters.cpp
)

foreach(TESTSRC ${TESTS})
    ADD_QGIS_TEST(${TESTSRC})
endforeach(TESTSRC)
