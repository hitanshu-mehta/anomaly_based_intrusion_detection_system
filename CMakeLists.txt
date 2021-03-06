cmake_minimum_required(VERSION 3.1.0)

# set project name
project(IDS)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -pthread")
set(CMAKE_BUILD_TYPE "Release")


include(InstallRequiredSystemLibraries)
set(CPACK_RESOURCE_FILE_LICENSE "${CMAKE_CURRENT_SOURCE_DIR}/LICENSE.md")
set(CPACK_PACKAGE_VERSION_MAJOR "${IDS_VERSION_MAJOR}")
set(CPACK_PACKAGE_VERSION_MINOR "${IDS_VERSION_MINOR}")


# Find includes in the build directories
set(CMAKE_INCLUDE_CURRENT_DIR ON)

set(LIBRARY_OUTPUT_PATH  ${EXECUTABLE_OUTPUT_PATH}/lib)
set(CMAKE_PREFIX_PATH ${CMAKE_SOURCE_DIR}/mongo-cxx-driver)
set(SRC_PATH ${CMAKE_SOURCE_DIR}/src)
set(SRC_HEADER_PATH ${CMAKE_SOURCE_DIR}/src/include)
set(UI_PATH ${SRC_PATH}/ui)
set(WORKER_PATH ${SRC_PATH}/worker)
set(GRAPH_PATH ${SRC_PATH}/graph)
set(MODEL_PATH ${CMAKE_SOURCE_DIR}/model)



#include(${CMAKE_BINARY_DIR}/conan.cmake)
#conan_cmake_run(CONANFILE conanfile.txt  # or relative build/conanfile.txt
#                BASIC_SETUP CMAKE_TARGETS
#                BUILD missing)


#add_subdirectory(pybind11)
include_directories(${CMAKE_SOURCE_DIR}/mongo-cxx-driver/include/mongocxx/v_noabi/)
include_directories(${CMAKE_SOURCE_DIR}/mongo-cxx-driver/include/bsoncxx/v_noabi/)
link_directories(${CMAKE_SOURCE_DIR}/mongo-cxx-driver/lib/)



list(APPEND CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/cmake)
# set(ENV{PYTHONPATH} ${CMAKE_SOURCE_DIR}/model/inference.py ${CMAKE_SOURCE_DIR}/model/fetch_data.py) 
find_package(Qt5 REQUIRED COMPONENTS Widgets Core Gui Charts)
find_package(pybind11 CONFIG REQUIRED)
find_package(PCAP REQUIRED)
find_package(mongocxx REQUIRED)
find_package(bsoncxx REQUIRED)


include(${CMAKE_BINARY_DIR}/conanbuildinfo.cmake)
conan_basic_setup()


set(project_source
    ${SRC_PATH}/main.cpp
    ${SRC_PATH}/mainwindow.cpp
    ${SRC_PATH}/packet_capture.cpp
    ${SRC_PATH}/packet_capture_engine.cpp
    ${SRC_PATH}/packet_parser.cpp
    ${SRC_PATH}/basic_flow.cpp
    ${SRC_PATH}/basic_packet_info.cpp
    ${SRC_PATH}/flow_generator.cpp
    ${SRC_PATH}/flow_generator_engine.cpp
    ${SRC_PATH}/database.cpp
    ${SRC_PATH}/session_stats.cpp
    ${WORKER_PATH}/packet_capture_worker.cpp
    ${WORKER_PATH}/ui_reload_worker.cpp
    ${WORKER_PATH}/inference_worker.cpp
    ${SRC_PATH}/Chart.cpp
    )

set(project_headers
    ${SRC_HEADER_PATH}/packet_capture.hpp
    ${SRC_HEADER_PATH}/packet_capture_engine.hpp
    ${SRC_HEADER_PATH}/packet_parser.hpp
    ${SRC_HEADER_PATH}/basic_flow.hpp
    ${SRC_HEADER_PATH}/basic_packet_info.hpp
    ${SRC_HEADER_PATH}/flow_features.hpp
    ${SRC_HEADER_PATH}/flow_generator.hpp
    ${SRC_HEADER_PATH}/flow_generator_engine.hpp
    ${SRC_HEADER_PATH}/constants.hpp
    ${SRC_HEADER_PATH}/ether_header.hpp
    ${SRC_HEADER_PATH}/ip_header.hpp
    ${SRC_HEADER_PATH}/protocols.hpp
    ${SRC_HEADER_PATH}/tcp_header.hpp
    ${SRC_HEADER_PATH}/udp_header.hpp
    ${SRC_HEADER_PATH}/utils.hpp
    ${SRC_HEADER_PATH}/session_stats.hpp
    ${SRC_HEADER_PATH}/mainwindow.h
    ${SRC_HEADER_PATH}/packet_capture_worker.h
    ${SRC_HEADER_PATH}/callback_timer.hpp
    ${SRC_HEADER_PATH}/ui_reload_worker.hpp
    ${SRC_HEADER_PATH}/inference_worker.hpp
    ${SRC_HEADER_PATH}/Chart.h
)

set (project_headers_mac ${SRC_HEADER_PATH}/mainwindow.h ${SRC_HEADER_PATH}/packet_capture_worker.h ${SRC_HEADER_PATH}/ui_reload_worker.hpp ${SRC_HEADER_PATH}/Chart.h)

set(project_ui ${UI_PATH}/mainwindow.ui)

qt5_wrap_ui(project_headers_wrapped ${project_ui})
qt5_wrap_cpp(project_source_mac ${project_headers_mac})

make_directory(${EXECUTABLE_OUTPUT_PATH}/bin/model)
file(COPY ${MODEL_PATH} DESTINATION ${CMAKE_BINARY_DIR}/bin)

add_executable(${PROJECT_NAME} ${project_source} ${project_headers} ${project_headers_wrapped} ${project_source_mac})



target_link_libraries(${PROJECT_NAME} ${CONAN_LIBS} ${PCAP_LIBRARY} mongocxx bsoncxx pybind11::embed Qt5::Widgets Qt5::Core Qt5::Gui Qt5::Charts)
