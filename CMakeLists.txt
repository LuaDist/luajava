# Copyright (C) 2007-2009 LuaDist.
# Created by Peter Kapec
# Redistribution and use of this file is allowed according to the terms of the MIT license.
# For details see the COPYRIGHT file distributed with LuaDist.
# Please note that the package source code is licensed under its own license.

PROJECT(luajava Java)
CMAKE_MINIMUM_REQUIRED(VERSION 2.6)
INCLUDE(dist.cmake)

# We need Java to compile
	FIND_PACKAGE (Java REQUIRED)
	FIND_PACKAGE (JNI REQUIRED)

	INCLUDE_DIRECTORIES(
#~ 		$ENV{JAVA_HOME}/include
#~ 		$ENV{JAVA_HOME}/include/win32
		${JAVA_INCLUDE_PATH}
		${JAVA_INCLUDE_PATH2}
	)
#~ 	SET(JAVA_BIN $ENV{JAVA_HOME}/bin)

	FIND_PROGRAM(JAVA_HEAD NAMES javah PATHS ${JAVA_BIN_PATH})


# LuaJava JAVA parts
	SET(VERSION 1.1)
	SET(PKG  luajava-${VERSION})
	SET(JAR_FILE ${PKG}.jar)
	
	FILE(GLOB JAVA_FILES ${CMAKE_CURRENT_SOURCE_DIR}/src/java/org/keplerproject/luajava/*.java)

	ADD_CUSTOM_TARGET(LUAJAVA_CLASSES
#~ 		COMMAND ${JAVA_BIN}/javac -sourcepath ${CMAKE_CURRENT_SOURCE_DIR}/src/java -d ${CMAKE_CURRENT_BINARY_DIR} ${CMAKE_CURRENT_SOURCE_DIR}/src/java/org/keplerproject/luajava/*.java

#~		COMMAND ${JAVA_COMPILE} -sourcepath ${CMAKE_CURRENT_SOURCE_DIR}/src/java -d ${CMAKE_CURRENT_BINARY_DIR} ${CMAKE_CURRENT_SOURCE_DIR}/src/java/org/keplerproject/luajava/*.java
		
COMMAND ${JAVA_COMPILE} -sourcepath ${CMAKE_CURRENT_SOURCE_DIR}/src/java -d ${CMAKE_CURRENT_BINARY_DIR}  ${JAVA_FILES}
		WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
		COMMENT	"Creating luajava classes"
		VERBATIM
	)
	
	FILE(GLOB JAVA_COMPILED_FILES ${CMAKE_CURRENT_BINARY_DIR}/org/keplerproject/luajava/*.class)	

	ADD_CUSTOM_TARGET(LUAJAVA_JAR
#~ 		COMMAND ${JAVA_BIN}/jar cvf ${JAR_FILE}  org/keplerproject/luajava/*.class
#~		COMMAND ${JAVA_ARCHIVE} cvf ${JAR_FILE}  org/keplerproject/luajava/*.class
		COMMAND ${JAVA_ARCHIVE} cvf ${JAR_FILE}  ${JAVA_COMPILED_FILES}
		WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
		COMMENT   "Creating luajava.jar"
		VERBATIM
	)
	ADD_DEPENDENCIES(LUAJAVA_JAR LUAJAVA_CLASSES)

	ADD_CUSTOM_TARGET(LUAJAVA_H
#~ 		COMMAND ${JAVA_BIN}/javah -o ${CMAKE_CURRENT_BINARY_DIR}/luajava.h -classpath ${CMAKE_CURRENT_BINARY_DIR} org.keplerproject.luajava.LuaState
		COMMAND ${JAVA_HEAD} -o ${CMAKE_CURRENT_BINARY_DIR}/luajava.h -classpath ${CMAKE_CURRENT_BINARY_DIR} org.keplerproject.luajava.LuaState
		WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
		COMMENT	"Creating luajava.h"
		VERBATIM
	)
	ADD_DEPENDENCIES(LUAJAVA_H LUAJAVA_JAR)


# LuaJava module
 	find_package(Lua51 REQUIRED)
	include_directories(${LUA_INCLUDE_DIR} ${CMAKE_CURRENT_BINARY_DIR})

	ADD_LIBRARY(luajava SHARED src/c/luajava.c)
	TARGET_LINK_LIBRARY(luajava ${LUA_LIBRARY})
	
	INSTALL(TARGETS luajava DESTINATION ${INSTALL_LIB})
	INSTALL(FILES ${CMAKE_CURRENT_BINARY_DIR}/${JAR_FILE} DESTINATION ${INSTALL_LIB} )

# LuaJava Batch & Shell scripts
	IF (WIN32 AND NOT CYGWIN AND NOT MSYS)
		SET(CONF_SUFFIX .bat)
	ENDIF ()

	CONFIGURE_FILE (${CMAKE_CURRENT_SOURCE_DIR}/luajava${CONF_SUFFIX}.cmake ${CMAKE_CURRENT_BINARY_DIR}/luajava${CONF_SUFFIX} @ONLY)
	INSTALL(FILES ${CMAKE_CURRENT_BINARY_DIR}/luajava${CONF_SUFFIX} DESTINATION ${INSTALL_BIN})

# Install docs and tests
	INSTALL(FILES License.txt DESTINATION ${INSTALL_DATA})
	INSTALL(DIRECTORY doc/ DESTINATION ${INSTALL_DOC})
	INSTALL(DIRECTORY test/ DESTINATION ${INSTALL_TEST})
