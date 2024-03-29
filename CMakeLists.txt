cmake_minimum_required(VERSION 3.15)

project(libxslt1 VERSION 1.1.34 LANGUAGES C)

list(APPEND CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR})

include(CheckCSourceCompiles)
include(CheckFunctionExists)
include(CheckIncludeFiles)
include(CheckLibraryExists)
include(CheckSymbolExists)
include(CMakePackageConfigHelpers)
include(GNUInstallDirs)


include_directories(
	BEFORE
    ../libxml2/include
	../lib/libxml2-lib
)




#find_package(LibXml2 CONFIG REQUIRED)

option(BUILD_SHARED_LIBS "Build shared libraries" ON)
option(LIBXSLT_WITH_DEBUGGER "Add the debugging support" ON)
option(LIBXSLT_WITH_CRYPTO "Add crypto support to exslt" OFF)
option(LIBXSLT_WITH_MEM_DEBUG "Add the memory debugging module" OFF)
option(LIBXSLT_WITH_MODULES "Add the module support" ON)
option(LIBXSLT_WITH_PROFILER "Add the profiling support" ON)
option(LIBXSLT_WITH_PYTHON "Build Python bindings" ON)
option(LIBXSLT_WITH_TESTS "Build tests" ON)
option(LIBXSLT_WITH_THREADS "Add multithread support" ON)
set(LIBXSLT_WITH_TRIO OFF)
option(LIBXSLT_WITH_XSLT_DEBUG "Add the debugging code" ON)

if(LIBXSLT_WITH_CRYPTO)
	find_package(Gcrypt REQUIRED)
endif()

if(LIBXSLT_WITH_PYTHON)
	check_include_files(unistd.h HAVE_UNISTD_H)
	check_symbol_exists(F_GETFL fcntl.h HAVE_F_GETFL)
	if(HAVE_UNISTD_H AND HAVE_F_GETFL)
		find_package(Python COMPONENTS Interpreter Development REQUIRED)
	else()
		find_package(Python2 COMPONENTS Interpreter Development REQUIRED)
		add_library(Python::Python ALIAS Python2::Python)
		set(Python_EXECUTABLE ${Python2_EXECUTABLE})
		set(Python_SITEARCH ${Python2_SITEARCH})
	endif()
	set(LIBXSLT_PYTHON_INSTALL_DIR ${Python_SITEARCH} CACHE PATH "Python bindings install directory")
endif()

if(LIBXSLT_WITH_THREADS)
	find_package(Threads REQUIRED)
endif()

foreach(VARIABLE IN ITEMS WITH_CRYPTO WITH_DEBUGGER WITH_MEM_DEBUG WITH_MODULES WITH_PROFILER WITH_PYTHON WITH_TRIO WITH_XSLT_DEBUG)
	if(LIBXSLT_${VARIABLE})
		set(${VARIABLE} 1)
	else()
		set(${VARIABLE} 0)
	endif()
endforeach()

set(LIBEXSLT_MAJOR_VERSION 0)
set(LIBEXSLT_MINOR_VERSION 8)
set(LIBEXSLT_MICRO_VERSION 20)

set(LIBEXSLT_DOTTED_VERSION ${LIBEXSLT_MAJOR_VERSION}.${LIBEXSLT_MINOR_VERSION}.${LIBEXSLT_MICRO_VERSION})
math(EXPR LIBEXSLT_VERSION_NUMBER "${LIBEXSLT_MAJOR_VERSION} * 10000 + ${LIBEXSLT_MINOR_VERSION} * 100 + ${LIBEXSLT_MICRO_VERSION}")
set(LIBEXSLT_VERSION_EXTRA "")

set(LIBXSLT_MAJOR_VERSION ${PROJECT_VERSION_MAJOR})
set(LIBXSLT_MINOR_VERSION ${PROJECT_VERSION_MINOR})
set(LIBXSLT_MICRO_VERSION ${PROJECT_VERSION_PATCH})

set(LIBXSLT_DOTTED_VERSION "${LIBXSLT_MAJOR_VERSION}.${LIBXSLT_MINOR_VERSION}.${LIBXSLT_MICRO_VERSION}")
math(EXPR LIBXSLT_VERSION_NUMBER "${LIBXSLT_MAJOR_VERSION} * 10000 + ${LIBXSLT_MINOR_VERSION} * 100 + ${LIBXSLT_MICRO_VERSION}")
set(LIBXSLT_VERSION_EXTRA "")

set(VERSION ${LIBXSLT_DOTTED_VERSION})
set(LIBEXSLT_VERSION ${LIBEXSLT_DOTTED_VERSION})

set(MODULE_EXTENSION "${CMAKE_SHARED_LIBRARY_SUFFIX}")

if(LIBXSLT_WITH_CRYPTO)
	list(APPEND CMAKE_REQUIRED_LIBRARIES Gcrypt::Gcrypt)
endif()

if(LIBXSLT_WITH_THREADS)
	list(APPEND CMAKE_REQUIRED_LIBRARIES Threads::Threads)
endif()

if(MSVC)
	configure_file(libxslt/win32config.h config.h COPYONLY)
	configure_file(libxslt/win32config.h win32config.h COPYONLY)
else()
	if(UNIX)
		set(CMAKE_REQUIRED_LIBRARIES m)
	endif()
	check_function_exists(clock_gettime HAVE_CLOCK_GETTIME)
	check_include_files(dlfcn.h HAVE_DLFCN_H)
	check_function_exists(fabs HAVE_FABS)
	check_include_files(float.h HAVE_FLOAT_H)
	check_function_exists(floor HAVE_FLOOR)
	check_function_exists(fprintf HAVE_FPRINTF)
	check_include_files(fp_class.h HAVE_FP_CLASS_H)
	check_function_exists(ftime HAVE_FTIME)
	check_library_exists(gcrypt gcry_control "gcrypt.h" HAVE_GCRYPT)
	check_function_exists(gettimeofday HAVE_GETTIMEOFDAY)
	check_function_exists(gmtime HAVE_GMTIME)
	check_function_exists(gmtime_r HAVE_GMTIME_R)
	check_include_files(ieeefp.h HAVE_IEEEFP_H)
	check_include_files(inttypes.h HAVE_INTTYPES_H)
	check_library_exists(pthread pthread_join "" HAVE_LIBPTHREAD)
	check_include_files(locale.h HAVE_LOCALE_H)
	check_function_exists(localtime HAVE_LOCALTIME)
	check_function_exists(localtime_r HAVE_LOCALTIME_R)
	check_include_files(math.h HAVE_MATH_H)
	check_include_files(memory.h HAVE_MEMORY_H)
	check_include_files(nan.h HAVE_NAN_H)
	check_function_exists(pow HAVE_POW)
	check_function_exists(printf HAVE_PRINTF)
	check_include_files(pthread.h HAVE_PTHREAD_H)
	check_function_exists(snprintf HAVE_SNPRINTF)
	check_function_exists(sprintf HAVE_SPRINTF)
	check_function_exists(sscanf HAVE_SSCANF)
	check_function_exists(stat HAVE_STAT)
	check_include_files(stdarg.h HAVE_STDARG_H)
	check_include_files(stdint.h HAVE_STDINT_H)
	check_include_files(stdlib.h HAVE_STDLIB_H)
	check_include_files(strings.h HAVE_STRINGS_H)
	check_include_files(string.h HAVE_STRING_H)
	check_function_exists(strxfrm_l HAVE_STRXFRM_L)
	check_include_files(sys/select.h HAVE_SYS_SELECT_H)
	check_include_files(sys/stat.h HAVE_SYS_STAT_H)
	check_include_files(sys/timeb.h HAVE_SYS_TIMEB_H)
	check_include_files(sys/time.h HAVE_SYS_TIME_H)
	check_include_files(sys/types.h HAVE_SYS_TYPES_H)
	check_function_exists(time HAVE_TIME)
	check_include_files(time.h HAVE_TIME_H)
	check_include_files(unistd.h HAVE_UNISTD_H)
	check_function_exists(vfprintf HAVE_VFPRINTF)
	check_function_exists(vsnprintf HAVE_VSNPRINTF)
	check_function_exists(vsprintf HAVE_VSPRINTF)
	check_include_files(xlocale.h HAVE_XLOCALE_H)
	check_function_exists(_stat HAVE__STAT)
	set(LT_OBJDIR ".libs/")
	set(PACKAGE "libxslt")
	set(PACKAGE_BUGREPORT "xml@gnome.org")
	set(PACKAGE_NAME "libxslt")
	set(PACKAGE_STRING "libxslt ${LIBXSLT_DOTTED_VERSION}")
	set(PACKAGE_TARNAME "libxslt")
	set(PACKAGE_URL "http://www.xmlsoft.org/libxslt")
	set(PACKAGE_VERSION ${LIBXSLT_DOTTED_VERSION})
	check_include_files("float.h;stdarg.h;stdlib.h;string.h" STDC_HEADERS)
	set(_ALL_SOURCE ON)
	set(_GNU_SOURCE ON)
	set(_POSIX_PTHREAD_SEMANTICS ON)
	set(_TANDEM_SOURCE ON)
	check_c_source_compiles("
		#define __EXTENSIONS__ 1
		int main() { return 0; }
	" __EXTENSIONS__)
	check_include_files(minix/config.h HAVE_MINIX_CONFIG_H)
	if(HAVE_MINIX_CONFIG_H)
		set(_MINIX ON)
		set(_POSIX_1_SOURCE ON)
		set(_POSIX_SOURCE ON)
	endif()
	configure_file(config.h.cmake.in config.h)
endif()

configure_file(libexslt/exsltconfig.h.in libexslt/exsltconfig.h)
configure_file(libxslt/xsltconfig.h.in libxslt/xsltconfig.h)

set(
	LIBXSLT_HDRS
	libxslt/attributes.h
	libxslt/documents.h
	libxslt/extensions.h
	libxslt/extra.h
	libxslt/functions.h
	libxslt/imports.h
	libxslt/keys.h
	libxslt/namespaces.h
	libxslt/numbersInternals.h
	libxslt/pattern.h
	libxslt/preproc.h
	libxslt/security.h
	libxslt/templates.h
	libxslt/transform.h
	libxslt/variables.h
	libxslt/xslt.h
	${CMAKE_CURRENT_BINARY_DIR}/libxslt/xsltconfig.h
	libxslt/xsltexports.h
	libxslt/xsltInternals.h
	libxslt/xsltlocale.h
	libxslt/xsltutils.h
)

set(
	LIBXSLT_SRCS
	libxslt/attributes.c
	libxslt/attrvt.c
	libxslt/documents.c
	libxslt/extensions.c
	libxslt/extra.c
	libxslt/functions.c
	libxslt/imports.c
	libxslt/keys.c
	libxslt/namespaces.c
	libxslt/numbers.c
	libxslt/pattern.c
	libxslt/preproc.c
	libxslt/security.c
	libxslt/templates.c
	libxslt/transform.c
	libxslt/variables.c
	libxslt/xslt.c
	libxslt/xsltlocale.c
	libxslt/xsltutils.c
)

add_library(LibXslt ${LIBXSLT_HDRS} ${LIBXSLT_SRCS})
add_library(LibXslt::LibXslt ALIAS LibXslt)

target_include_directories(
	LibXslt
	PUBLIC
	$<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}>
	$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
	$<INSTALL_INTERFACE:$<INSTALL_PREFIX>/${CMAKE_INSTALL_INCLUDEDIR}>
	
)

link_directories(
    ../lib/libxml2-lib
)

target_link_libraries(LibXslt PUBLIC xml2)

if(UNIX)
	target_link_libraries(LibXslt PRIVATE m)
	set(M_LIBS "-lm")
endif()

set_target_properties(
	LibXslt
	PROPERTIES
	IMPORT_PREFIX lib
	OUTPUT_NAME xslt
	POSITION_INDEPENDENT_CODE ON
	PREFIX lib
	VERSION ${PROJECT_VERSION}
)

if(MSVC)
	if(BUILD_SHARED_LIBS)
		set_target_properties(
			LibXslt
			PROPERTIES
			DEBUG_POSTFIX d
		)
	else()
		set_target_properties(
			LibXslt
			PROPERTIES
			DEBUG_POSTFIX sd
			MINSIZEREL_POSTFIX s
			RELEASE_POSTFIX s
			RELWITHDEBINFO_POSTFIX s
		)
	endif()
endif()

install(FILES ${LIBXSLT_HDRS} DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/libxslt COMPONENT development)

install(
	TARGETS	LibXslt
	EXPORT LibXslt
	ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR} COMPONENT development
	LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR} COMPONENT runtime NAMELINK_COMPONENT development
	RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR} COMPONENT runtime
)

if(MSVC AND BUILD_SHARED_LIBS)
	install(FILES $<TARGET_PDB_FILE:LibXslt> DESTINATION ${CMAKE_INSTALL_BINDIR} CONFIGURATIONS Debug RelWithDebInfo COMPONENT debug)
endif()

set(
	LIBEXSLT_HDRS
	libexslt/exslt.h
	${CMAKE_CURRENT_BINARY_DIR}/libexslt/exsltconfig.h
	libexslt/exsltexports.h
)

set(
	LIBEXSLT_SRCS
	libexslt/common.c
	libexslt/crypto.c
	libexslt/date.c
	libexslt/dynamic.c
	libexslt/exslt.c
	libexslt/functions.c
	libexslt/libexslt.h
	libexslt/math.c
	libexslt/saxon.c
	libexslt/sets.c
	libexslt/strings.c
)

add_library(LibExslt ${LIBEXSLT_HDRS} ${LIBEXSLT_SRCS})
add_library(LibXslt::LibExslt ALIAS LibExslt)

target_include_directories(
	LibExslt
	PUBLIC
	$<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}>
	$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
	$<INSTALL_INTERFACE:$<INSTALL_PREFIX>/${CMAKE_INSTALL_INCLUDEDIR}>
)

if(LIBXSLT_WITH_CRYPTO)
	target_link_libraries(LibExslt PRIVATE Gcrypt::Gcrypt)
	set(LIBGCRYPT_LIBS "-lgcrypt")
endif()

target_link_libraries(LibExslt PUBLIC LibXslt xml2)

set_target_properties(
	LibExslt
	PROPERTIES
	IMPORT_PREFIX lib
	OUTPUT_NAME exslt
	POSITION_INDEPENDENT_CODE ON
	PREFIX lib
	VERSION ${LIBEXSLT_VERSION}
)

if(MSVC)
	if(BUILD_SHARED_LIBS)
		set_target_properties(
			LibExslt
			PROPERTIES
			DEBUG_POSTFIX d
		)
	else()
		set_target_properties(
			LibExslt
			PROPERTIES
			DEBUG_POSTFIX sd
			MINSIZEREL_POSTFIX s
			RELEASE_POSTFIX s
			RELWITHDEBINFO_POSTFIX s
		)
	endif()
endif()

install(FILES ${LIBEXSLT_HDRS} DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/libexslt COMPONENT development)

install(
	TARGETS	LibExslt
	EXPORT LibXslt
	ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR} COMPONENT development
	LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR} COMPONENT runtime NAMELINK_COMPONENT development
	RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR} COMPONENT runtime
)

if(MSVC AND BUILD_SHARED_LIBS)
	install(FILES $<TARGET_PDB_FILE:LibExslt> DESTINATION ${CMAKE_INSTALL_BINDIR} CONFIGURATIONS Debug RelWithDebInfo COMPONENT debug)
endif()

add_executable(xsltproc xsltproc/xsltproc.c)
add_executable(LibXslt::xsltproc ALIAS xsltproc)
target_include_directories(xsltproc PRIVATE $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>)
target_link_libraries(xsltproc LibExslt LibXslt)
install(TARGETS xsltproc EXPORT LibXslt RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR} COMPONENT programs)

if(LIBXSLT_WITH_TESTS)
	enable_testing()
	if(Threads_FOUND)
		add_executable(testThreadsXslt xsltproc/testThreads.c)
		target_link_libraries(testThreadsXslt LibXslt LibExslt Threads::Threads)
		add_test(NAME testThreadsXslt COMMAND testThreadsXslt WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR})
	endif()
endif()

if(LIBXSLT_WITH_PYTHON)
	set(ENV{SRCDIR} ${CMAKE_CURRENT_SOURCE_DIR}/python)
	execute_process(
		COMMAND
		${Python_EXECUTABLE}
		${CMAKE_CURRENT_SOURCE_DIR}/python/generator.py
		${CMAKE_CURRENT_SOURCE_DIR}/doc/libxslt-api.xml
		${CMAKE_CURRENT_SOURCE_DIR}/python/libxslt-python-api.xml
		WORKING_DIRECTORY
		${CMAKE_CURRENT_BINARY_DIR}
	)
	unset(ENV{SRCDIR})
	file(READ python/libxsl.py LIBXSL_PY)
	file(WRITE ${CMAKE_CURRENT_BINARY_DIR}/libxslt.py.in "${LIBXSL_PY}")
	file(READ ${CMAKE_CURRENT_BINARY_DIR}/libxsltclass.py LIBXSLTCLASS_PY)
	file(APPEND ${CMAKE_CURRENT_BINARY_DIR}/libxslt.py.in "${LIBXSLTCLASS_PY}")
	configure_file(${CMAKE_CURRENT_BINARY_DIR}/libxslt.py.in libxslt.py COPYONLY)
	add_library(
		LibXsltMod
		libxslt-py.c
		libxslt-py.h
		python/libxml_wrap.h
		python/libxslt.c
		python/libxslt_wrap.h
		python/types.c
	)
	target_include_directories(
		LibXsltMod
		PUBLIC
		$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/python>
	)
	target_link_libraries(LibXsltMod LibXslt LibExslt Python::Python)
	set_target_properties(
		LibXsltMod
		PROPERTIES
		IMPORT_PREFIX lib
		OUTPUT_NAME xsltmod
		PREFIX lib
		VERSION ${PROJECT_VERSION}
	)
	install(
		TARGETS	LibXsltMod
		ARCHIVE DESTINATION ${LIBXSLT_PYTHON_INSTALL_DIR} COMPONENT development
		LIBRARY DESTINATION ${LIBXSLT_PYTHON_INSTALL_DIR} COMPONENT runtime NAMELINK_COMPONENT development
		RUNTIME DESTINATION ${LIBXSLT_PYTHON_INSTALL_DIR} COMPONENT runtime
	)
	if(MSVC AND BUILD_SHARED_LIBS)
		install(FILES $<TARGET_PDB_FILE:LibXsltMod> DESTINATION ${LIBXSLT_PYTHON_INSTALL_DIR} CONFIGURATIONS Debug RelWithDebInfo COMPONENT debug)
	endif()
	install(FILES ${CMAKE_CURRENT_BINARY_DIR}/libxslt.py DESTINATION ${LIBXSLT_PYTHON_INSTALL_DIR} COMPONENT runtime)
endif()

install(FILES libexslt/libexslt.3 DESTINATION ${CMAKE_INSTALL_MANDIR}/man3 COMPONENT documentation)
install(FILES libxslt/libxslt.3 DESTINATION ${CMAKE_INSTALL_MANDIR}/man3 COMPONENT documentation)
install(FILES doc/xsltproc.1 DESTINATION ${CMAKE_INSTALL_MANDIR}/man1 COMPONENT documentation)
install(DIRECTORY doc/ DESTINATION ${CMAKE_INSTALL_DATADIR}/doc/libxslt COMPONENT documentation PATTERN Makefile.* EXCLUDE)

if(LIBXSLT_WITH_CRYPTO)
	install(FILES FindGcrypt.cmake DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/libxslt-${PROJECT_VERSION} COMPONENT development)
endif()

configure_package_config_file(
	libxslt-config.cmake.cmake.in libxslt-config.cmake
	INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/libxslt-${PROJECT_VERSION}
)

install(
	FILES ${CMAKE_CURRENT_BINARY_DIR}/libxslt-config.cmake
	DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/libxslt-${PROJECT_VERSION}
	COMPONENT development
)

write_basic_package_version_file(
	${CMAKE_CURRENT_BINARY_DIR}/libxslt-config-version.cmake
	VERSION ${PROJECT_VERSION}
	COMPATIBILITY ExactVersion
)

install(
	FILES ${CMAKE_CURRENT_BINARY_DIR}/libxslt-config-version.cmake
	DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/libxslt-${PROJECT_VERSION}
	COMPONENT development
)

install(
	EXPORT LibXslt
	DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/libxslt-${PROJECT_VERSION}
	NAMESPACE LibXslt::
	FILE libxslt-export.cmake
	COMPONENT development
)

set(LIBXSLT_DEFAULT_PLUGINS_PATH "\${libdir}/libxslt-plugins")
set(XSLT_INCLUDEDIR "-I\${includedir}")
set(XSLT_LIBDIR "-L\${libdir}")
set(XSLT_LIBS "-lxslt -lxml2")
set(XSLT_PRIVATE_LIBS "${M_LIBS}")

set(EXSLT_INCLUDEDIR "-I\${includedir}")
set(EXSLT_LIBDIR "-L\${libdir}")
set(EXSLT_PRIVATE_LIBS "${XSLT_PRIVATE_LIBS} ${LIBGCRYPT_LIBS}")

file(RELATIVE_PATH PACKAGE_RELATIVE_PATH "${CMAKE_INSTALL_FULL_LIBDIR}/pkgconfig" "${CMAKE_INSTALL_PREFIX}")
string(REGEX REPLACE "/$" "" PACKAGE_RELATIVE_PATH "${PACKAGE_RELATIVE_PATH}")

set(prefix "\${pcfiledir}/${PACKAGE_RELATIVE_PATH}")
set(exec_prefix "\${prefix}")
set(libdir "\${prefix}/${CMAKE_INSTALL_LIBDIR}")
set(includedir "\${prefix}/${CMAKE_INSTALL_INCLUDEDIR}")
configure_file(libxslt.pc.in libxslt.pc @ONLY)
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/libxslt.pc DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig COMPONENT development)
configure_file(libexslt.pc.in libexslt.pc @ONLY)
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/libexslt.pc DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig COMPONENT development)

set(prefix "\$(cd \"\$(dirname \"\$0\")\"; pwd -P)/..")
configure_file(xslt-config.in xslt-config @ONLY)
install(PROGRAMS ${CMAKE_CURRENT_BINARY_DIR}/xslt-config DESTINATION ${CMAKE_INSTALL_BINDIR} COMPONENT development)

set(XSLT_INCLUDEDIR "-I${CMAKE_INSTALL_FULL_INCLUDEDIR}")
set(XSLT_LIBDIR "-L${CMAKE_INSTALL_FULL_LIBDIR}")
configure_file(xsltConf.sh.in xsltConf.sh @ONLY)
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/xsltConf.sh DESTINATION ${CMAKE_INSTALL_LIBDIR} COMPONENT development)

set(CPACK_COMPONENT_DEVELOPMENT_DEPENDS runtime)
set(CPACK_COMPONENT_PROGRAMS_DEPENDS runtime)
set(CPACK_DEB_COMPONENT_INSTALL ON)
set(CPACK_DEBIAN_DEVELOPMENT_PACKAGE_DEPENDS "${PACKAGE_TARNAME}")
set(CPACK_DEBIAN_DEVELOPMENT_PACKAGE_NAME "${PACKAGE_TARNAME}${LIBXSLT_MAJOR_VERSION}-dev")
set(CPACK_DEBIAN_DEVELOPMENT_PACKAGE_SECTION "libdevel")
set(CPACK_DEBIAN_PACKAGE_HOMEPAGE ${PACKAGE_URL})
set(CPACK_DEBIAN_PACKAGE_NAME ${PACKAGE_TARNAME})
set(CPACK_DEBIAN_PACKAGE_SECTION "devel")
set(CPACK_DEBIAN_PROGRAMS_PACKAGE_DEPENDS "${PACKAGE_TARNAME}")
set(CPACK_DEBIAN_PROGRAMS_PACKAGE_NAME "${PACKAGE_TARNAME}-utils")
set(CPACK_DEBIAN_PROGRAMS_PACKAGE_SECTION "utils")
set(CPACK_DEBIAN_RUNTIME_PACKAGE_NAME ${PACKAGE_TARNAME})
set(CPACK_DEBIAN_RUNTIME_PACKAGE_RECOMMENDS "${PACKAGE_TARNAME}-utils")
set(CPACK_DEBIAN_RUNTIME_PACKAGE_SECTION "libs")
set(CPACK_NSIS_PACKAGE_NAME ${PACKAGE_STRING})
set(CPACK_NSIS_URL_INFO_ABOUT ${PACKAGE_URL})
set(CPACK_PACKAGE_CONTACT ${PACKAGE_BUGREPORT})
set(CPACK_PACKAGE_DISPLAY_NAME ${PACKAGE_STRING})
set(CPACK_PACKAGE_INSTALL_DIRECTORY "${PACKAGE_TARNAME}-${PACKAGE_VERSION}")
set(CPACK_PACKAGE_NAME ${PACKAGE_TARNAME})
set(CPACK_PACKAGE_VERSION ${PACKAGE_VERSION})
set(CPACK_PACKAGE_VERSION_MAJOR ${LIBXSLT_MAJOR_VERSION})
set(CPACK_PACKAGE_VERSION_MINOR ${LIBXSLT_MINOR_VERSION})
set(CPACK_PACKAGE_VERSION_PATCH ${LIBXSLT_MICRO_VERSION})
set(CPACK_RESOURCE_FILE_LICENSE ${CMAKE_CURRENT_SOURCE_DIR}/Copyright)
set(CPACK_RPM_COMPONENT_INSTALL ON)
set(CPACK_RPM_development_PACKAGE_NAME "${PACKAGE_NAME}-devel")
set(CPACK_RPM_development_PACKAGE_REQUIRES "${PACKAGE_NAME}")
set(CPACK_RPM_PACKAGE_GROUP "Development/Libraries")
set(CPACK_RPM_PACKAGE_NAME ${PACKAGE_TARNAME})
set(CPACK_RPM_PACKAGE_URL ${PACKAGE_URL})
set(CPACK_RPM_programs_PACKAGE_NAME "${PACKAGE_NAME}-utils")
set(CPACK_RPM_programs_PACKAGE_REQUIRES "${PACKAGE_NAME}")
set(CPACK_RPM_runtime_PACKAGE_NAME "${PACKAGE_NAME}")
set(CPACK_RPM_runtime_PACKAGE_SUGGESTS "${PACKAGE_NAME}-utils")

include(CPack)
