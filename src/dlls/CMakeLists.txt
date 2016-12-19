###################################################
#                                                 #
#                                                 #
#   RCBot CMake build file                        #
#                                                 #
#                                                 #
###################################################

#Change this to match your plugin's directory structure.
set( PLUGIN_LIB_DIR ${META_BASE_DIRECTORY}/rcbot/dlls )

set( PLUGIN_NAME rcbot )

#Set plugin specific linker flags here.
set( PLUGIN_LINK_FLAGS )

set( BUILD_TYPE CACHE STRING "Build type" )
set_property( CACHE BUILD_TYPE PROPERTY STRINGS Metamod Standalone )

#Metamod builds define RCBOT_META_BUILD
if( BUILD_TYPE STREQUAL "Metamod" )
	set( RCBOT_IS_METAMOD 1 )
	set( RCBOT_META_BUILD "RCBOT_META_BUILD" )
else()
	set( RCBOT_IS_METAMOD 0 )
	set( RCBOT_META_BUILD )
endif()

if( NOT RCBOT_IS_METAMOD )
	#Generate or update linkgame.cpp
	MESSAGE( STATUS "Generating linkgame.cpp..." )

	if( WIN32 )
		#Check if Perl is installed.
		find_program( PERL perl )
		
		if( NOT PERL )
			MESSAGE( FATAL_ERROR "Perl must be installed in order to run mklinkgame.pl\nSee https://www.perl.org" )
		endif()
		
		set( MKLINKGAME_COMMAND perl )
		set( MKLINKGAME_ARGS "mklinkgame.pl" )
	else()
		set( MKLINKGAME_COMMAND )
		set( MKLINKGAME_ARGS "./mklinkgame.sh" )
	endif()

	#The command and arguments need to be separate in order for the command itself to be picked up correctly.
	execute_process( COMMAND ${MKLINKGAME_COMMAND} ${MKLINKGAME_ARGS}
		WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
		OUTPUT_VARIABLE MKLINKGAME_OUTPUT
		ERROR_VARIABLE MKLINKGAME_OUTPUT
	)

	MESSAGE( STATUS ${MKLINKGAME_OUTPUT} )

	MESSAGE( STATUS "Done." )
else()
	MESSAGE( STATUS "Not generating linkgame.cpp due to Metamod build" )
endif()

#Add plugin sources here, or using add_subdirectory.
add_sources(
	${SHARED_SOURCES}
	actionutility.h
	activity.h
	archtypes.h
	basemonster.h
	bits.cpp
	bits.h
	bot.cpp
	bot.h
	bot_builder.h
	bot_client.cpp
	bot_client.h
	bot_commands.cpp
	bot_commands.h
	bot_const.h
	bot_enemy_cost_ga.h
	bot_func.h
	bot_ga.cpp
	bot_ga.h
	bot_learnsys.h
	bot_menu.cpp
	bot_menu.h
	bot_navigate.cpp
	bot_nn.cpp
	bot_rep.cpp
	bot_texture.h
	bot_tick.h
	bot_visibles.cpp
	bot_visibles.h
	bot_weapons.cpp
	bot_weapons.h
	cbase.h
	cdll_dll.h
	client.cpp
	dir.cpp
	dir.h
	dll.cpp
	engine.cpp
	engine.h
	enginecallback.h
	extdll.h
	ga.cpp
	ga.h
	generic_class.h
	GiveFnptrsToDllExport.h
	globals.cpp
	h_export.cpp
	h_export.h
	linkent.cpp
	megahal.cpp
	megahal.h
	monsterevent.h
	netmessages.cpp
	nn.h
	perceptron.cpp
	perceptron.h
	player.h
	rcbot_meta_api.cpp
	saverestore.h
	schedule.h
	som.cpp
	som.h
	studio.h
	usercmd.h
	util.cpp
	util.h
	vector.h
	waypoint.cpp
	waypoint.h
)

#Only include these files if it's not a Metamod build, since it handles some things differently.
if( NOT RCBOT_IS_METAMOD )
add_sources(
	linkgame.cpp
)
endif()

#Add public plugin headers here, if any.
#add_includes(
#
#)

#Process source files for inclusion.
preprocess_sources()

#Add text files related to the project. Set directly to the prepared sources to avoid VS filters from putting it in dlls/.
set( PREP_SRCS
	${PREP_SRCS}
	${CMAKE_CURRENT_SOURCE_DIR}/notes.txt
	${CMAKE_CURRENT_SOURCE_DIR}/todo.txt
	${CMAKE_CURRENT_SOURCE_DIR}/version.txt
)

add_library( ${PLUGIN_NAME} SHARED ${PREP_SRCS} )

#Add include paths here.
target_include_directories( ${PLUGIN_NAME} PRIVATE
	.
	${SHARED_INCLUDE_PATHS}
)

#Define preprocessor symbols here.
target_compile_definitions( ${PLUGIN_NAME} PRIVATE
	${SHARED_DEFINITIONS}
	${RCBOT_META_BUILD}
	VALVE_DLL
)

#Add library dependencies here.
target_link_libraries( ${PLUGIN_NAME}
	${SHARED_LIBRARY_DEPS}
)

#If the user wants automatic deployment to a game directory, set the output directory paths.
if( DEPLOY_TO_GAME )
	#CMake places libraries in /Debug or /Release on Windows, so explicitly set the paths for both.
	#On Linux, it uses LIBRARY_OUTPUT_DIRECTORY
	set_target_properties( ${PLUGIN_NAME} PROPERTIES
		LIBRARY_OUTPUT_DIRECTORY ${PLUGIN_LIB_DIR}
		RUNTIME_OUTPUT_DIRECTORY_DEBUG ${PLUGIN_LIB_DIR}
		RUNTIME_OUTPUT_DIRECTORY_RELEASE ${PLUGIN_LIB_DIR}
	)
endif()

#Set 32 bit flag, any plugin specific flags.
set_target_properties( ${PLUGIN_NAME} 
	PROPERTIES COMPILE_FLAGS "${LINUX_32BIT_FLAG}" 
	LINK_FLAGS "${SHARED_LINKER_FLAGS} ${PLUGIN_LINK_FLAGS} ${LINUX_32BIT_FLAG}"
)

#No lib prefix.
SET_TARGET_PROPERTIES( ${PLUGIN_NAME} PROPERTIES PREFIX "" )

#Output current build type on build
add_custom_command( TARGET ${PLUGIN_NAME} 
	PRE_BUILD
	COMMAND echo Build type: ${BUILD_TYPE}
)

#Create filters.
create_source_groups( "${CMAKE_SOURCE_DIR}" )

#Clear sources list for next target.
clear_sources()
