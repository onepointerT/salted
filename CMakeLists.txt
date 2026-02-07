
cmake_minimum_required( VERSION 3.15 )

project( salted C CXX )

set( LIBSALTED_STABLE_VERSION 0 )
set( LIBSALTED_RELEASE 0 )
set( LIBSALTED_PATCH_VERSION 1 )
set( LIBSALTED_VERSION "${LIBSALTED_STABLE_VERSION}.${LIBSALTED_RELEASE}.${LIBSALTED_PATCH_VERSION}" )

set( LIBSALTED_SRC_DIR ${PROJECT_SOURCE_DIR}/src )

set ( PROJECT_NAME "${CMAKE_PROJECT_NAME}" )
set ( PROJECT_VERSION "${LIBSALTED_VERSION}" )
set ( PROJECT_DESCRIPTION "A desktop gui API basing on CSS and jademl." )

# Build the jademl extensions
add_subdirectory( lib/jademl )
include_directories( lib/jademl/include )
# Build submodule elements
if( _MSC_BUILD )
	set(ELEMENTS_HOST_UI_LIBRARY "win32")
else()
	set(ELEMENTS_HOST_UI_LIBRARY "gtk")
endif()
#add_subdirectory( lib/elements )
include_directories( lib/elements/lib/include )
# Build the lunasvg/css painting extension
#add_subdirectory( lib/lunasvg )
include_directories( lib/lunasvg/include lib/lunasvg/source )
# Build the openGL graphics backend pantheon
include_directories( lib/nuklearcpp/lib/gui/pantheon/lib lib/nuklearcpp/lib/gui/pantheon/src )
# Build the UI backend nuklearcpp
add_subdirectory( lib/nuklearcpp )
include_directories( lib/nuklearcpp/include )
# Build the database engine
add_subdirectory( lib/sqlitecpp )
include_directories( lib/sqlitecpp/include )

if( SALTED_DESTDIR )
else()
	set( SALTED_DESTDIR "${CMAKE_INSTALL_PREFIX}" )
endif()


file ( GLOB LIBSALTED_SOURCE ${LIBSALTED_SRC_DIR}/*.cpp )

if( SALTED_SHARED )
	add_library( salted SHARED ${LIBSALTED_SOURCE} )
else()
	add_library( salted STATIC ${LIBSALTED_SOURCE} )
endif()
#target_link_libraries( salted  )


if( SALTED_HEADER_INSTALL_DIR )
else()
	set( SALTED_HEADER_INSTALL_DIR ${SALTED_DESTDIR}/include/LIBSALTED )
endif()

if ( SALTED_LIB_INSTALL_DIR )
else()
	set ( SALTED_LIB_INSTALL_DIR ${SALTED_DESTDIR}/lib )
endif()

if( SALTED_INSTALL_LIBS )
	install(TARGETS SALTED DESTINATION ${SALTED_LIB_INSTALL_DIR} )
	install(FILES ${LIBSALTED_HEADERS} DESTINATION ${SALTED_HEADER_INSTALL_DIR} )
	install(FILES ${PROJECT_SOURCE_DIR}/lib/LIBSALTEDConfig.cmake DESTINATION "${SALTED_LIB_INSTALL_DIR}/cmake/LIBSALTED" )
	configure_file(${PROJECT_SOURCE_DIR}/lib/LIBSALTED.pc.in ${CMAKE_CURRENT_BINARY_DIR}/LIBSALTED.pc @ONLY)
	install(FILES ${CMAKE_CURRENT_BINARY_DIR}/LIBSALTED.pc DESTINATION "${SALTED_LIB_INSTALL_DIR}/pkgconfig" )
endif()


if( UNIX AND ENABLE_DOC_SALTED )
	find_package(Doxygen)
	if(DOXYGEN_FOUND)
		message("-- Enabled doc_SALTED target for make")
		configure_file(${CMAKE_CURRENT_SOURCE_DIR}/Doxyfile.in ${CMAKE_CURRENT_BINARY_DIR}/Doxyfile @ONLY)
		add_custom_target(doc_SALTED ${DOXYGEN_EXECUTABLE} ${CMAKE_CURRENT_BINARY_DIR}/Doxyfile WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR} COMMENT "Generating documentation with Doxygen")
	endif()
endif()
