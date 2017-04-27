if(NOT HAVE_PYTHON)
  message(STATUS "Python interface is disabled or not all required dependencies found. Building without it...")
  return()
endif()

include_directories(${PYTHON_INCLUDE_DIRS} ${NUMPY_INCLUDE_DIR} ${Boost_INCLUDE_DIRS})
file(GLOB_RECURSE python_srcs ${PROJECT_SOURCE_DIR}/python/*.cpp)

set(CMAKE_SHARED_LINKER_FLAGS "-L/usr/local/lib -Wl,-Bstatic -lprotobuf -Wl,-Bdynamic -pthread -lpthread ${CMAKE_SHARED_LINKER_FLAGS}")
add_library(pycaffe SHARED ${python_srcs})
target_link_libraries(pycaffe ${Caffe_LINK} ${PYTHON_LIBRARIES} ${Boost_LIBRARIES})
set_target_properties(pycaffe PROPERTIES PREFIX "" OUTPUT_NAME "_caffe8")
caffe_default_properties(pycaffe)

if(UNIX OR APPLE)
    set(__linkname "${PROJECT_SOURCE_DIR}/python/caffe8/_caffe8.so")
    add_custom_command(TARGET pycaffe POST_BUILD
                       COMMAND ln -sf $<TARGET_LINKER_FILE:pycaffe> "${__linkname}"
                       COMMAND ${CMAKE_COMMAND} -E make_directory ${PROJECT_SOURCE_DIR}/python/caffe8/proto
                       COMMAND touch ${PROJECT_SOURCE_DIR}/python/caffe8/proto/__init__.py
                       COMMAND cp ${proto_gen_folder}/*.py ${PROJECT_SOURCE_DIR}/python/caffe8/proto/
                       COMMENT "Creating symlink ${__linkname} -> ${PROJECT_BINARY_DIR}/lib/_caffe8${Caffe_POSTFIX}.so")
endif()

# ---[ Install
# scripts
file(GLOB python_files *.py requirements.txt)
install(FILES ${python_files} DESTINATION python)

# module
install(DIRECTORY caffe/
    DESTINATION python/caffe8
    FILES_MATCHING
    PATTERN "*.py"
    PATTERN "ilsvrc_2012_mean.npy"
    PATTERN "test" EXCLUDE
    )

# _caffe.so
install(TARGETS pycaffe  DESTINATION python/caffe8)

