#
# project: fips-soloud
#
cmake_minimum_required(VERSION 2.8)

# include the fips main cmake file
get_filename_component(FIPS_ROOT_DIR "../fips" ABSOLUTE)
include("${FIPS_ROOT_DIR}/cmake/fips.cmake")

fips_setup()
    fips_project(fips-soloud)

    set (CMAKE_MODULE_PATH "${PROJECT_SOURCE_DIR}/soloud/contrib/cmake;${CMAKE_MODULE_PATH}")
    # include("${PROJECT_SOURCE_DIR}/soloud/contrib/cmake/FindSDL2.cmake")

    option (SOLOUD_DYNAMIC "Set to ON to build dynamic SoLoud" OFF)
    option (SOLOUD_STATIC "Set to ON to build static SoLoud" ON)
    option (SOLOUD_BACKEND_NULL "Set to ON for building NULL backend" ON)
    option (SOLOUD_BACKEND_SDL2 "Set to ON for building SDL2 backend" ON)
    option (SOLOUD_BACKEND_COREAUDIO "Set to ON for building CoreAudio backend" OFF)
    option (SOLOUD_BACKEND_OPENSLES "Set to ON for building OpenSLES backend" OFF)
    option (SOLOUD_BACKEND_XAUDIO2 "Set to ON for building XAudio2 backend" OFF)
    option (SOLOUD_BACKEND_WINMM "Set to ON for building WINMM backend" OFF)
    option (SOLOUD_BACKEND_WASAPI "Set to ON for building WASAPI backend" OFF)

    if (${SOLOUD_DYNAMIC})
        fips_begin_sharedlib(soloud)
    elseif(${SOLOUD_STATIC})
        fips_begin_lib(Soloud)
    endif()

        # fips_ide_group(Imports/soloud)
        fips_include_directories(soloud/include)

        fips_src(soloud/include GROUP "includes" NO_RECURSE)
        fips_src(soloud/src/core GROUP "core" NO_RECURSE)
        fips_src(soloud/src/audiosource GROUP "audiosources")
        fips_src(soloud/src/filter GROUP "filters" NO_RECURSE)

        set (SOURCE_PATH soloud/src)

        # Backends
        set (BACKENDS_PATH ${SOURCE_PATH}/backend)
        set (BACKENDS_SOURCES)

        if (SOLOUD_BACKEND_NULL)
            set (BACKENDS_SOURCES
                ${BACKENDS_SOURCES}
                ${BACKENDS_PATH}/null/soloud_null.cpp
            )
            add_definitions(-DWITH_NULL)
        endif()

        if (SOLOUD_BACKEND_SDL2)
            find_package (SDL2 REQUIRED)
            include_directories (${SDL2_INCLUDE_DIR})
            add_definitions (-DWITH_SDL2_STATIC)

            set (BACKENDS_SOURCES
                ${BACKENDS_SOURCES}
                ${BACKENDS_PATH}/sdl2_static/soloud_sdl2_static.cpp
            )

            set (LINK_LIBRARIES
                ${LINK_LIBRARIES}
                ${SDL2_LIBRARY}
            )

        endif()

        if (SOLOUD_BACKEND_COREAUDIO)
            if (NOT APPLE)
                message (FATAL_ERROR "CoreAudio backend can be enabled only on Apple!")
            endif ()

            add_definitions (-DWITH_COREAUDIO)

            set (BACKENDS_SOURCES
                ${BACKENDS_SOURCES}
                ${BACKENDS_PATH}/coreaudio/soloud_coreaudio.cpp
            )

            find_library (AUDIOTOOLBOX_FRAMEWORK AudioToolbox)
            set (LINK_LIBRARIES
                ${LINK_LIBRARIES}
                ${AUDIOTOOLBOX_FRAMEWORK}
            )
        endif()


        if (SOLOUD_BACKEND_OPENSLES)
            add_definitions (-DWITH_OPENSLES)

            set (BACKENDS_SOURCES
                ${BACKENDS_SOURCES}
                ${BACKENDS_PATH}/opensles/soloud_opensles.cpp
            )

            find_library (OPENSLES_LIBRARY OpenSLES)
            set (LINK_LIBRARIES
                ${LINK_LIBRARIES}
                ${OPENSLES_LIBRARY}
            )
        endif()


        if (SOLOUD_BACKEND_XAUDIO2)
            add_definitions (-DWITH_XAUDIO2)

            set (BACKENDS_SOURCES
                ${BACKENDS_SOURCES}
                ${BACKENDS_PATH}/xaudio2/soloud_xaudio2.cpp
            )
        endif()

        if (SOLOUD_BACKEND_WINMM)
            add_definitions (-DWITH_WINMM)

            set (BACKENDS_SOURCES
                ${BACKENDS_SOURCES}
                ${BACKENDS_PATH}/winmm/soloud_winmm.cpp
            )
        endif()
    
        if (SOLOUD_BACKEND_WASAPI)
            add_definitions (-DWITH_WASAPI)

            set (BACKENDS_SOURCES
                ${BACKENDS_SOURCES}
                ${BACKENDS_PATH}/wasapi/soloud_wasapi.cpp
            )
        endif()

        message(STATUS "${BACKENDS_SOURCES}")

        fips_dir(.)
        fips_files(${BACKENDS_SOURCES})

    if (${SOLOUD_DYNAMIC})
        fips_end_sharedlib()
    elseif(${SOLOUD_STATIC})
        fips_end_lib()
    endif()

fips_finish()


