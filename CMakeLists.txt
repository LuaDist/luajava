# Copyright (C) 2007-2012 LuaDist.
# Created by Peter Kapec
# Redistribution and use of this file is allowed according to the terms of the MIT license.
# For details see the COPYRIGHT file distributed with LuaDist.
# Please note that the package source code is licensed under its own license.

project ( luajava Java )
cmake_minimum_required ( VERSION 2.8 )
include ( cmake/dist.cmake )
include ( lua )

# We need Java to compile
find_package ( Java REQUIRED )
find_package ( JNI REQUIRED )

include_directories ( #~ 		$ENV{JAVA_HOME}/include
  #~ 		$ENV{JAVA_HOME}/include/win32
  ${JAVA_INCLUDE_PATH} ${JAVA_INCLUDE_PATH2} )
#~ 	SET(JAVA_BIN $ENV{JAVA_HOME}/bin)

find_program ( JAVA_HEAD NAMES javah PATHS ${JAVA_BIN_PATH} )

# LuaJava JAVA parts
set ( VERSION 1.1 )
set ( PKG luajava-${VERSION} )
set ( JAR_FILE ${PKG}.jar )
file ( GLOB JAVA_FILES ${CMAKE_CURRENT_SOURCE_DIR}/src/java/org/keplerproject/luajava/*.java )

add_custom_target ( LUAJAVA_CLASSES #~ 		COMMAND ${JAVA_BIN}/javac -sourcepath ${CMAKE_CURRENT_SOURCE_DIR}/src/java -d ${CMAKE_CURRENT_BINARY_DIR} ${CMAKE_CURRENT_SOURCE_DIR}/src/java/org/keplerproject/luajava/*.java
  

  #~		COMMAND ${JAVA_COMPILE} -sourcepath ${CMAKE_CURRENT_SOURCE_DIR}/src/java -d ${CMAKE_CURRENT_BINARY_DIR} ${CMAKE_CURRENT_SOURCE_DIR}/src/java/org/keplerproject/luajava/*.java
  COMMAND ${JAVA_COMPILE} -sourcepath ${CMAKE_CURRENT_SOURCE_DIR}/src/java -d ${CMAKE_CURRENT_BINARY_DIR} 
  ${JAVA_FILES} WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR} COMMENT "Creating luajava classes" 
  VERBATIM )
file ( GLOB JAVA_COMPILED_FILES ${CMAKE_CURRENT_BINARY_DIR}/org/keplerproject/luajava/*.class )

add_custom_target ( LUAJAVA_JAR #~ 		COMMAND ${JAVA_BIN}/jar cvf ${JAR_FILE}  org/keplerproject/luajava/*.class
  #~		COMMAND ${JAVA_ARCHIVE} cvf ${JAR_FILE}  org/keplerproject/luajava/*.class
  COMMAND ${JAVA_ARCHIVE} cvf ${JAR_FILE} ${JAVA_COMPILED_FILES} WORKING_DIRECTORY 
  ${CMAKE_CURRENT_BINARY_DIR} COMMENT "Creating luajava.jar" VERBATIM )
add_dependencies ( LUAJAVA_JAR LUAJAVA_CLASSES )

add_custom_target ( LUAJAVA_H #~ 		COMMAND ${JAVA_BIN}/javah -o ${CMAKE_CURRENT_BINARY_DIR}/luajava.h -classpath ${CMAKE_CURRENT_BINARY_DIR} org.keplerproject.luajava.LuaState
  COMMAND ${JAVA_HEAD} -o ${CMAKE_CURRENT_BINARY_DIR}/luajava.h -classpath ${CMAKE_CURRENT_BINARY_DIR} 
  org.keplerproject.luajava.LuaState WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR} 
  COMMENT "Creating luajava.h" VERBATIM )
add_dependencies ( LUAJAVA_H LUAJAVA_JAR )

# LuaJava module
find_package ( Lua51 REQUIRED )
include_directories ( ${LUA_INCLUDE_DIR} ${CMAKE_CURRENT_BINARY_DIR} )

add_library ( luajava src/c/luajava.c )
target_link_library ( luajava ${LUA_LIBRARY} )
install_library ( luajava )
install ( FILES ${CMAKE_CURRENT_BINARY_DIR}/${JAR_FILE} DESTINATION ${INSTALL_LIB} COMPONENT Library )

# LuaJava Batch & Shell scripts
if ( WIN32 AND NOT CYGWIN AND NOT MSYS )
  set ( CONF_SUFFIX .bat )
endif ( )

configure_file ( ${CMAKE_CURRENT_SOURCE_DIR}/luajava${CONF_SUFFIX}.cmake ${CMAKE_CURRENT_BINARY_DIR}/luajava${CONF_SUFFIX} 
  @ONLY )
install_executable ( ${CMAKE_CURRENT_BINARY_DIR}/luajava${CONF_SUFFIX} )

# Install docs and tests
install_data ( License.txt )
install_doc ( doc/ )
install_test ( test/ )
