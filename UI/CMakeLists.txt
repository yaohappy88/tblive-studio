option(ENABLE_UI "Enables the OBS user interfaces" ON)
if(DISABLE_UI)
	message(STATUS "UI disabled")
	return()
elseif(ENABLE_UI)
	set(FIND_MODE REQUIRED)
else()
	set(FIND_MODE QUIET)
endif()

add_subdirectory(obs-frontend-api)

# ----------------------------------------------------------------------------

project(TBLiveStudio)

set(ENABLE_WIN_UPDATER FALSE CACHE BOOL "Enable the windows updater")

if(DEFINED QTDIR${_lib_suffix})
	list(APPEND CMAKE_PREFIX_PATH "${QTDIR${_lib_suffix}}")
elseif(DEFINED QTDIR)
	list(APPEND CMAKE_PREFIX_PATH "${QTDIR}")
elseif(DEFINED ENV{QTDIR${_lib_suffix}})
	list(APPEND CMAKE_PREFIX_PATH "$ENV{QTDIR${_lib_suffix}}")
elseif(DEFINED ENV{QTDIR})
	list(APPEND CMAKE_PREFIX_PATH "$ENV{QTDIR}")
endif()

set(CMAKE_INCLUDE_CURRENT_DIR TRUE)
set(CMAKE_AUTOMOC TRUE)

if(CMAKE_SIZEOF_VOID_P EQUAL 8)
	set(_lib_suffix 64)
else()
	set(_lib_suffix 32)
endif()

find_package(Qt5Widgets ${FIND_MODE})
find_package(Qt5WebEngine ${FIND_MODE})
find_package(Qt5WebEngineWidgets ${FIND_MODE})
find_package(Qt5WebChannel ${FIND_MODE})
find_package(FFmpeg REQUIRED COMPONENTS avcodec avutil avformat)

if(NOT Qt5Widgets_FOUND)
	if (ENABLE_UI)
		message(FATAL_ERROR "Failed to find Qt5")
	else()
		message(STATUS "Qt5 not found - UI disabled")
		return()
	endif()
endif()


include_directories(${FFMPEG_INCLUDE_DIRS})
include_directories(SYSTEM "obs-frontend-api")
include_directories(SYSTEM "${CMAKE_SOURCE_DIR}/libobs")
include_directories(SYSTEM "${CMAKE_SOURCE_DIR}/deps/libff")
include_directories(SYSTEM "${CMAKE_SOURCE_DIR}/deps/tblivesdk/include")

find_package(Libcurl REQUIRED)
include_directories(${LIBCURL_INCLUDE_DIRS})
add_definitions(${LIBCURL_DEFINITIONS})

if(WIN32)
	include_directories(${OBS_JANSSON_INCLUDE_DIRS})
	include_directories(${BLAKE2_INCLUDE_DIR})

	set(obs_PLATFORM_SOURCES
		platform-windows.cpp
		win-update/update-window.cpp
		win-update/win-update.cpp
		win-update/win-update-helpers.cpp
		obs.rc)
	set(obs_PLATFORM_HEADERS
		win-update/update-window.hpp
		win-update/win-update.hpp
		win-update/win-update-helpers.hpp
		crash/win-mini-dump.hpp)
	set(obs_PLATFORM_LIBRARIES
		${CMAKE_SOURCE_DIR}/deps/tblivesdk/lib/Release/TBLiveSDK.lib
		crypt32
		blake2
		${OBS_JANSSON_IMPORT})

	if(ENABLE_WIN_UPDATER)
		add_definitions(-DENABLE_WIN_UPDATER)
	endif()
elseif(APPLE)
	set(obs_PLATFORM_SOURCES
		platform-osx.mm)

	find_package(AppKit REQUIRED)
	set(obs_PLATFORM_LIBRARIES ${APPKIT_LIBRARIES})

	add_definitions(-fobjc-arc)

	option(ENABLE_SPARKLE_UPDATER "Enables updates via the Sparkle framework (don't forget to update the Info.plist for your .app)" OFF)
	if(ENABLE_SPARKLE_UPDATER)
		find_library(SPARKLE Sparkle)
		include_directories(${SPARKLE})
		set(obs_PLATFORM_SOURCES
			${obs_PLATFORM_SOURCES}
			sparkle-updater.mm)
		set(obs_PLATFORM_LIBRARIES
			${obs_PLATFORM_LIBRARIES}
			${SPARKLE})
		add_definitions(-DUPDATE_SPARKLE=1)
	endif()

elseif(UNIX)
        find_package(Qt5X11Extras REQUIRED)

	set(obs_PLATFORM_SOURCES
		platform-x11.cpp)

        set(obs_PLATFORM_LIBRARIES
                Qt5::X11Extras)
endif()

set(obs_libffutil_SOURCES
	../deps/libff/libff/ff-util.c
	)
set(obs_libffutil_HEADERS
	../deps/libff/libff/ff-util.h
	)

if(MSVC)
	set_source_files_properties(
		../deps/libff/libff/ff-util.c
		PROPERTIES COMPILE_FLAGS -Dinline=__inline
		)
endif()

set(obs_SOURCES
	${obs_PLATFORM_SOURCES}
	${obs_libffutil_SOURCES}
	obs-app.cpp
	api-interface.cpp
	window-basic-end.cpp
	window-basic-main.cpp
	window-basic-stats.cpp
	window-basic-filters.cpp
	window-basic-settings.cpp
	window-basic-interaction.cpp
	window-basic-properties.cpp
	window-basic-main-outputs.cpp
	window-basic-source-select.cpp
	window-basic-startpage.cpp
	window-basic-main-scene-collections.cpp
	window-basic-main-transitions.cpp
	window-basic-main-dropfiles.cpp
	window-basic-main-profiles.cpp
	window-basic-main-tblive.cpp
	window-license-agreement.cpp
	window-basic-status-bar.cpp
	window-basic-adv-audio.cpp
	window-basic-transform.cpp
	window-basic-preview.cpp
	window-namedialog.cpp
	window-log-reply.cpp
	window-projector.cpp
	window-remux.cpp
	properties-view.cpp
	focus-list.cpp
	menu-button.cpp
	double-slider.cpp
	volume-control.cpp
	adv-audio-control.cpp
	item-widget-helpers.cpp
	visibility-checkbox.cpp
	locked-checkbox.cpp
	vertical-scroll-area.cpp
	visibility-item-widget.cpp
	slider-absoluteset-style.cpp
	source-list-widget.cpp
	qt-display.cpp
	crash-report.cpp
	hotkey-edit.cpp
	source-label.cpp
	remote-text.cpp
	audio-encoders.cpp
	qt-wrappers.cpp
	stream-url-dialog.cpp
	window-hoverwidget.cpp
	resolution-dock-widget.cpp
	stream-state-window.cpp
	volume-control-custom.cpp)

set(obs_HEADERS
	${obs_PLATFORM_HEADERS}
	${obs_libffutil_HEADERS}
	obs-app.hpp
	platform.hpp
	window-main.hpp
	window-basic-end.h
	window-basic-main.hpp
	window-basic-stats.hpp
	window-basic-filters.hpp
	window-basic-settings.hpp
	window-basic-interaction.hpp
	window-basic-properties.hpp
	window-basic-main-outputs.hpp
	window-basic-source-select.hpp
	window-basic-startpage.hpp
	window-license-agreement.hpp
	window-basic-status-bar.hpp
	window-basic-adv-audio.hpp
	window-basic-transform.hpp
	window-basic-preview.hpp
	window-namedialog.hpp
	window-log-reply.hpp
	window-projector.hpp
	window-remux.hpp
	properties-view.hpp
	properties-view.moc.hpp
	display-helpers.hpp
	double-slider.hpp
	focus-list.hpp
	menu-button.hpp
	mute-checkbox.hpp
	volume-control.hpp
	adv-audio-control.hpp
	item-widget-helpers.hpp
	visibility-checkbox.hpp
	locked-checkbox.hpp
	vertical-scroll-area.hpp
	visibility-item-widget.hpp
	slider-absoluteset-style.hpp
	source-list-widget.hpp
	qt-display.hpp
	crash-report.hpp
	hotkey-edit.hpp
	source-label.hpp
	remote-text.hpp
	audio-encoders.hpp
	qt-wrappers.hpp
	commonToolFun.hpp
	stream-url-dialog.h
	window-hoverwidget.hpp
	resolution-dock-widget.hpp
	stream-state-window.h
	source-type-defines.hpp
	volume-control-custom.hpp)

set(obs_UI
	forms/NameDialog.ui
	forms/OBSLicenseAgreement.ui
	forms/OBSLogReply.ui
	forms/OBSBasic.ui
	forms/OBSBasicEnd.ui
	forms/OBSBasicTransform.ui
	forms/OBSBasicFilters.ui
	forms/OBSBasicSettings.ui
	forms/OBSBasicSourceSelect.ui
	forms/OBSBasicStartPage.ui
	forms/OBSBasicInteraction.ui
	forms/OBSUpdate.ui
	forms/HoverWidget.ui
	forms/ResolutionDockWidget.ui
	forms/StreamUrlDialog.ui
	forms/OBSRemux.ui)

set(obs_QRC
	forms/obs.qrc)

qt5_wrap_ui(obs_UI_HEADERS ${obs_UI})
qt5_add_resources(obs_QRC_SOURCES ${obs_QRC})

add_executable(TBLiveStudio WIN32
	${obs_SOURCES}
	${obs_HEADERS}
	${obs_UI_HEADERS}
	${obs_QRC_SOURCES})

if(WIN32)
	if(CMAKE_SIZEOF_VOID_P EQUAL 8)
		set(_output_suffix "64")
	else()
		set(_output_suffix "32")
	endif()

	set_target_properties(TBLiveStudio
		PROPERTIES
			OUTPUT_NAME "TBLiveStudio")
endif()

target_link_libraries(TBLiveStudio
	libobs
	Qt5::Widgets
	Qt5::WebEngine
    Qt5::WebEngineWidgets
    Qt5::WebChannel
	obs-frontend-api
	${FFMPEG_LIBRARIES}
	${LIBCURL_LIBRARIES}
	${obs_PLATFORM_LIBRARIES})


define_graphic_modules(TBLiveStudio)

install_obs_core(TBLiveStudio)
install_obs_data(TBLiveStudio data tblive-studio)

if (UNIX AND UNIX_STRUCTURE AND NOT APPLE)
	install(FILES dist/obs.desktop DESTINATION ${CMAKE_INSTALL_FULL_DATAROOTDIR}/applications)
	install(FILES forms/icons/icon.png
		DESTINATION ${CMAKE_INSTALL_FULL_DATAROOTDIR}/icons/hicolor/256x256/apps)
endif()

add_subdirectory(frontend-plugins)
if(WIN32)
	add_subdirectory(win-update/updater)
endif()
