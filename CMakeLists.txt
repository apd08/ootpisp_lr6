cmake_minimum_required(VERSION 3.16)
project(LibrarySerialization LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Shared core: hierarchy + factory + plugin manager (single copy for app and plugins).
add_library(library_core SHARED
    src/LibraryObject.cpp
    src/Publication.cpp
    src/Book.cpp
    src/Magazine.cpp
    src/FictionBook.cpp
    src/Textbook.cpp
    src/Newspaper.cpp
    src/ObjectFactory.cpp
    src/PluginManager.cpp
)

target_include_directories(library_core PUBLIC include)
target_compile_definitions(library_core PRIVATE LIBRARY_CORE_BUILDING)

add_executable(library_app
    src/main.cpp
    src/ConsoleUI.cpp
    src/XmlSerializer.cpp
)

target_include_directories(library_app PRIVATE include)
target_link_libraries(library_app PRIVATE library_core)

set(PLUGIN_OUTPUT_DIR "${CMAKE_BINARY_DIR}/plugins")

# Lab 4: hierarchy plugin (ComicBook type).
add_library(comic_plugin SHARED
    plugins/comic_plugin/ComicBook.cpp
)
target_include_directories(comic_plugin PRIVATE include plugins/comic_plugin)
target_link_libraries(comic_plugin PRIVATE library_core)
set_target_properties(comic_plugin PROPERTIES
    RUNTIME_OUTPUT_DIRECTORY "${PLUGIN_OUTPUT_DIR}"
    LIBRARY_OUTPUT_DIRECTORY "${PLUGIN_OUTPUT_DIR}"
    ARCHIVE_OUTPUT_DIRECTORY "${PLUGIN_OUTPUT_DIR}"
)

# Lab 5: XML transform plugins (variant 4).
add_library(xml_indent_plugin SHARED
    plugins/xml_indent_plugin/XmlIndentPlugin.cpp
)
target_include_directories(xml_indent_plugin PRIVATE include)
target_link_libraries(xml_indent_plugin PRIVATE library_core)
set_target_properties(xml_indent_plugin PROPERTIES
    RUNTIME_OUTPUT_DIRECTORY "${PLUGIN_OUTPUT_DIR}"
    LIBRARY_OUTPUT_DIRECTORY "${PLUGIN_OUTPUT_DIR}"
    ARCHIVE_OUTPUT_DIRECTORY "${PLUGIN_OUTPUT_DIR}"
)

add_library(xml_comment_plugin SHARED
    plugins/xml_comment_plugin/XmlCommentPlugin.cpp
)
target_include_directories(xml_comment_plugin PRIVATE include)
target_link_libraries(xml_comment_plugin PRIVATE library_core)
set_target_properties(xml_comment_plugin PROPERTIES
    RUNTIME_OUTPUT_DIRECTORY "${PLUGIN_OUTPUT_DIR}"
    LIBRARY_OUTPUT_DIRECTORY "${PLUGIN_OUTPUT_DIR}"
    ARCHIVE_OUTPUT_DIRECTORY "${PLUGIN_OUTPUT_DIR}"
)

add_library(xslt_plugin SHARED
    plugins/xslt_plugin/XsltPlugin.cpp
)
target_include_directories(xslt_plugin PRIVATE include)
target_link_libraries(xslt_plugin PRIVATE library_core)
if(WIN32)
    target_link_libraries(xslt_plugin PRIVATE ole32 oleaut32)
endif()
set_target_properties(xslt_plugin PROPERTIES
    RUNTIME_OUTPUT_DIRECTORY "${PLUGIN_OUTPUT_DIR}"
    LIBRARY_OUTPUT_DIRECTORY "${PLUGIN_OUTPUT_DIR}"
    ARCHIVE_OUTPUT_DIRECTORY "${PLUGIN_OUTPUT_DIR}"
)

# Copy XSLT stylesheets next to built plugins for runtime use.
file(MAKE_DIRECTORY "${PLUGIN_OUTPUT_DIR}")
configure_file(
    "${CMAKE_SOURCE_DIR}/plugins/xslt_plugin/library_lab5.xslt"
    "${PLUGIN_OUTPUT_DIR}/library_lab5.xslt"
    COPYONLY
)
configure_file(
    "${CMAKE_SOURCE_DIR}/plugins/xslt_plugin/library_strip_lab.xslt"
    "${PLUGIN_OUTPUT_DIR}/library_strip_lab.xslt"
    COPYONLY
)

if(MSVC)
    target_compile_options(library_core PRIVATE /W4)
    target_compile_options(library_app PRIVATE /W4)
    target_compile_options(comic_plugin PRIVATE /W4)
    target_compile_options(xml_indent_plugin PRIVATE /W4)
    target_compile_options(xml_comment_plugin PRIVATE /W4)
    target_compile_options(xslt_plugin PRIVATE /W4)
else()
    target_compile_options(library_core PRIVATE -Wall -Wextra -Wpedantic)
    target_compile_options(library_app PRIVATE -Wall -Wextra -Wpedantic)
    target_compile_options(comic_plugin PRIVATE -Wall -Wextra -Wpedantic)
    target_compile_options(xml_indent_plugin PRIVATE -Wall -Wextra -Wpedantic)
    target_compile_options(xml_comment_plugin PRIVATE -Wall -Wextra -Wpedantic)
    target_compile_options(xslt_plugin PRIVATE -Wall -Wextra -Wpedantic)
endif()
