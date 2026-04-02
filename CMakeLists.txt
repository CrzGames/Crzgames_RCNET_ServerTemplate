# CMake 3.28.0 est la version minimale requise pour ce projet
cmake_minimum_required(VERSION 3.28.0)

# ============================================================
# Project Configuration
# ============================================================
# Nom du projet : rcnet-server-template
#
# Langages :
#   - C     : Certaines dépendences
#   - CXX   : C++ pour le moteur de serveur jeu (RCNET) et les 
#             fonctionnalités principales
# ============================================================
project(rcnet-server-template
  LANGUAGES C CXX
)

# ============================================================
# Language Standards
# ============================================================
# CXX : RCNET est écrit en C++ 17.
#
# C : requis uniquement pour : nats client c..etc
# ============================================================
set(CMAKE_C_STANDARD 99)
set(CMAKE_C_STANDARD_REQUIRED ON)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# ============================================================
# Output Directories
# ============================================================
# Gestion du dossier de sortie selon le générateur CMake :
# - Visual Studio / Xcode : multi-config (Debug/Release/...)
# - Ninja / Makefiles     : single-config
# ============================================================
if(CMAKE_GENERATOR MATCHES "Visual Studio|Xcode")
  # Multi-config generators
  set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/$<CONFIGURATION>")
  set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/$<CONFIGURATION>")
else()
  # Single-config generators (Makefiles, Ninja, etc.)
  set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}")
  set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}")
endif()

# ============================================================
# Build Options RCNET (Feature toggles)
# ============================================================
# Toutes les options RCNET_* qui permettent d'activer/désactiver
# des modules, features ou intégrations SDK.
# ============================================================

# Toujours désactivé dans un server template
set(RCNET_BUILD_EXAMPLE OFF CACHE BOOL "" FORCE)
set(RCNET_BUILD_TESTS   OFF CACHE BOOL "" FORCE)

# Le reste : défauts, mais la CI peut override avec -D
set(RCNET_AGONES_ENABLED ON CACHE BOOL "")

# ============================================================
# Compiler Warnings & Strictness
# ============================================================
# Flags d'avertissements et de conformité :
# - MSVC : /W4, /permissive- (C++ strict), désactivations ciblées
# - GCC/Clang : -Wall -Wextra -pedantic + exceptions ciblées
# ============================================================
function(rcnet_server_template_apply_warnings target_name)
  target_compile_options(${target_name} PRIVATE
    # MSVC
    # /W4 : Niveau d'avertissement 4 (equivalent a -Wextra)
    # /permissive- : Spécifie un mode de compilation strict pour suivre les standards C++ 
    # /wd4100 : Désactive l'avertissement pour les paramètres de fonction non utilisés (C4100)
    "$<$<COMPILE_LANG_AND_ID:C,MSVC>:/W4;/wd4100>"
    "$<$<COMPILE_LANG_AND_ID:CXX,MSVC>:/W4;/permissive-;/wd4100>"

    # GCC / Clang / AppleClang - C
    # -Wall : Active tous les avertissements de base
    # -Wextra : Active des avertissements supplémentaires
    # -pedantic : Enforce strictement les standards C++
    # -Wno-unused-parameter : Désactive les avertissements pour les paramètres de fonction non utilisés
    # -Wno-error=strict-prototypes : Désactive les erreurs pour les prototypes de fonction stricts en C.
    "$<$<COMPILE_LANG_AND_ID:C,GNU,Clang,AppleClang>:-Wall;-Wextra;-pedantic;-Wno-unused-parameter;-Wno-error=strict-prototypes>"

    # GCC / Clang / AppleClang - C++
    "$<$<COMPILE_LANG_AND_ID:CXX,GNU,Clang,AppleClang>:-Wall;-Wextra;-pedantic;-Wno-unused-parameter>"

    # Clang-specific: newline at end of file
    # Clang a un warning spécifique pour les fichiers qui ne se terminent pas par une nouvelle ligne (newline at end of file).
    # -Wno-newline-eof : Désactive les avertissements pour les nouvelles lignes à la fin du fichier
    "$<$<COMPILE_LANG_AND_ID:C,Clang,AppleClang>:-Wno-newline-eof>"
    "$<$<COMPILE_LANG_AND_ID:CXX,Clang,AppleClang>:-Wno-newline-eof>"
  )
endfunction()

# ============================================================
# Dépendances vendored — add_subdirectory
# ============================================================
# On inclut les dépendances vendored (clonées via setup_dependencies.cmake) 
# en utilisant add_subdirectory.
# ============================================================
add_subdirectory(dependencies/Crzgames_RCNET EXCLUDE_FROM_ALL)

# ============================================================
# Build Environment (dev / staging / production)
# ============================================================
set(SERVER_ENV "dev" CACHE STRING "Build environment: dev, staging or production")
set_property(CACHE SERVER_ENV PROPERTY STRINGS "dev" "staging" "production")
if(NOT SERVER_ENV STREQUAL "dev"
   AND NOT SERVER_ENV STREQUAL "staging"
   AND NOT SERVER_ENV STREQUAL "production")
  message(FATAL_ERROR "SERVER_ENV invalide: '${SERVER_ENV}' (attendu: dev, staging ou production)")
endif()

# ============================================================
# rcnet_target_compile_definitions
# ============================================================
function(rcnet_target_compile_definitions target_name)
  # Environement de build (dev/staging/production) : ajoute des macros préprocesseur pour différencier les builds
  # dans le code source, par exemple pour activer des logs de debug en dev mais pas en production.
  if(SERVER_ENV STREQUAL "dev")
    target_compile_definitions(${target_name} PRIVATE
      SERVER_ENV_DEV=1
      SERVER_ENV_STAGING=0
      SERVER_ENV_PRODUCTION=0
    )
  elseif(SERVER_ENV STREQUAL "staging")
    target_compile_definitions(${target_name} PRIVATE
      SERVER_ENV_DEV=0
      SERVER_ENV_STAGING=1
      SERVER_ENV_PRODUCTION=0
    )
  else() # production
    target_compile_definitions(${target_name} PRIVATE
      SERVER_ENV_DEV=0
      SERVER_ENV_STAGING=0
      SERVER_ENV_PRODUCTION=1
    )
  endif()
endfunction()

# ============================================================
# Sources & Headers (RCNET)
# ============================================================
# Récupère automatiquement tous les fichiers sources (.cpp) et 
# d'en-tête (.h) du projet pour les ajouter à la target.
# ============================================================

# Ajouter les fichiers sources du server de jeu.
file(GLOB_RECURSE SOURCES 
  "${PROJECT_SOURCE_DIR}/src/*.cpp"
)

# Ajouter les fichiers d'en-tête du server de jeu.
file(GLOB_RECURSE HEADERS 
  "${PROJECT_SOURCE_DIR}/include/*.h"
)

# ============================================================
# Target principale du projet
# ============================================================
# Crée la target exécutable principale du projet en incluant 
# les sources et les headers.
# ============================================================
add_executable(${PROJECT_NAME}
  ${SOURCES}
  ${HEADERS}
)

# Apply compiler warnings to the main target
rcnet_server_template_apply_warnings(${PROJECT_NAME})

# ============================================================
# Include Directories
# ============================================================
target_include_directories(${PROJECT_NAME} PRIVATE
  "${PROJECT_SOURCE_DIR}/include"
)

# ============================================================
# Compile Definitions
# ============================================================
rcnet_target_compile_definitions(${PROJECT_NAME})

# ============================================================
# Linker avec la bibliothèque RCNET
#
# Lie la target à la bibliothèque RCNET pour pouvoir utiliser le moteur
# de serveur.
# ============================================================
target_link_libraries(${PROJECT_NAME} PRIVATE
  rcnet
)
