cmake_minimum_required(VERSION 3.25)

project(GameBoostPro VERSION 0.1.0 LANGUAGES CXX)

if(NOT WIN32)
    message(FATAL_ERROR "GameBoost Pro supports Windows 10/11 x64 only.")
endif()

if(NOT CMAKE_SIZEOF_VOID_P EQUAL 8)
    message(FATAL_ERROR "GameBoost Pro requires a 64-bit toolchain.")
endif()

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)

find_package(Qt6 6.6 CONFIG REQUIRED COMPONENTS
    Core
    Gui
    Qml
    Quick
    Charts
    LinguistTools
)
find_package(unofficial-sqlite3 CONFIG REQUIRED)
find_package(unofficial-sodium CONFIG REQUIRED)
find_package(nlohmann_json CONFIG REQUIRED)
find_package(spdlog CONFIG REQUIRED)

# Shared dependency target for application targets added by later modules.
add_library(gameboost_pro_dependencies INTERFACE)
target_link_libraries(gameboost_pro_dependencies INTERFACE
    Qt6::Core
    Qt6::Gui
    Qt6::Qml
    Qt6::Quick
    Qt6::Charts
    unofficial::sqlite3::sqlite3
    unofficial-sodium::sodium
    nlohmann_json::nlohmann_json
    spdlog::spdlog
    advapi32
    iphlpapi
    psapi
)

# Call for each Windows executable so the elevation manifest is embedded.
function(gameboost_apply_app_manifest target)
    if(NOT TARGET "${target}")
        message(FATAL_ERROR "gameboost_apply_app_manifest: '${target}' is not a target.")
    endif()

    get_target_property(_gameboost_target_type "${target}" TYPE)
    if(NOT _gameboost_target_type STREQUAL "EXECUTABLE")
        message(FATAL_ERROR
            "gameboost_apply_app_manifest: '${target}' must be an executable target."
        )
    endif()

    if(MSVC)
        set(_gameboost_manifest "${PROJECT_SOURCE_DIR}/app.manifest")
        target_link_options("${target}" PRIVATE
            "/MANIFEST:EMBED"
            "/MANIFESTINPUT:${_gameboost_manifest}"
        )
        set_property(TARGET "${target}" PROPERTY VS_GLOBAL_VCManifestFile
            "${_gameboost_manifest}"
        )
    else()
        message(FATAL_ERROR
            "Embedding app.manifest is currently configured for MSVC-compatible linkers."
        )
    endif()
endfunction()
