cmake_minimum_required(VERSION 3.0)
project(win-phone-app)

if(MSVC_VERSION GREATER 1700)
  set(COMPILER_VERSION "12")
elseif(MSVC_VERSION GREATER 1600)
  set(COMPILER_VERSION "11")
endif()

set (APP_MANIFEST_NAME Package.appxmanifest)

if("${CMAKE_SYSTEM_NAME}" STREQUAL "WindowsPhone")
  set(PLATFORM WP)
  add_definitions("-DPHONE")
  if("${CMAKE_SYSTEM_VERSION}" STREQUAL "8.0")
    set(APP_MANIFEST_NAME WMAppManifest.xml)
    set(WINDOWS_PHONE8 1)
  endif()
else()
  set(PLATFORM DESKTOP)
  message(FATAL_ERROR "This app supports Store / Phone only. Please edit the target platform.")
endif()

set_property(GLOBAL PROPERTY USE_FOLDERS ON)

set(EXE_NAME win-phone-app)
set(SHORT_NAME ${EXE_NAME})
set(PACKAGE_GUID "6514377e-dfd4-4cdb-80df-4e0366346efc")

if (NOT "${PLATFORM}" STREQUAL "DESKTOP")
  configure_file(
    cmake/Package_vc${COMPILER_VERSION}.${PLATFORM}.appxmanifest.in
    ${CMAKE_CURRENT_BINARY_DIR}/${APP_MANIFEST_NAME}
    @ONLY)
endif()

set(SOURCE_FILES
  app.cpp
  pch.cpp
  win-appMain.cpp
  DeviceResources.cpp
  Sample3DSceneRenderer.cpp
  SampleFpsTextRenderer.cpp
  )

set(HEADER_FILES
  app.h
  pch.h
  win-appMain.h
  DeviceResources.h
  DirectXHelper.h
  StepTimer.h
  ShaderStructures.h
  Sample3DSceneRenderer.h
  SampleFpsTextRenderer.h
  )

set(PIXELSHADER_FILES
  SamplePixelShader.hlsl
  )

set(VERTEXSHADER_FILES
  SampleVertexShader.hlsl
  )

set(CONTENT_FILES ${PIXELSHADER_FILES} ${VERTEXSHADER_FILES})

if (WINDOWS_PHONE8)
  set(CONTENT_FILES ${CONTENT_FILES}
    ${CMAKE_CURRENT_BINARY_DIR}/${APP_MANIFEST_NAME}
    Assets/Logo.scale-240.png
    Assets/SmallLogo.scale-240.png
    Assets/SplashScreen.scale-240.png
    Assets/Square71x71Logo.scale-240.png
    Assets/StoreLogo.scale-240.png
    Assets/WideLogo.scale-240.png
    )
  # Windows Phone 8.0 needs to copy all the images.
  # It doesn't know to use relative paths.
  file(COPY
    Assets/Logo.scale-240.png
    Assets/SmallLogo.scale-240.png
    Assets/SplashScreen.scale-240.png
    Assets/Square71x71Logo.scale-240.png
    Assets/StoreLogo.scale-240.png
    Assets/WideLogo.scale-240.png
    DESTINATION ${CMAKE_CURRENT_BINARY_DIR}
    )
endif()

#set(RESOURCE_FILES
#  ${CONTENT_FILES} ${DEBUG_CONTENT_FILES} ${RELEASE_CONTENT_FILES} ${ASSET_FILES}
#  win-phone-app_TemporaryKey.pfx)

set_property(SOURCE ${CONTENT_FILES} PROPERTY VS_DEPLOYMENT_CONTENT 1)
set_property(SOURCE ${ASSET_FILES} PROPERTY VS_DEPLOYMENT_CONTENT 1)
set_property(SOURCE ${ASSET_FILES} PROPERTY VS_DEPLOYMENT_LOCATION "Assets")
set_property(SOURCE ${DEBUG_CONTENT_FILES} PROPERTY VS_DEPLOYMENT_CONTENT $<CONFIG:Debug>)
set_property(SOURCE ${RELEASE_CONTENT_FILES} PROPERTY VS_DEPLOYMENT_CONTENT $<CONFIG:Release>)

set_property(SOURCE ${PIXELSHADER_FILES} PROPERTY VS_SHADER_TYPE Pixel)
set_property(SOURCE ${PIXELSHADER_FILES} PROPERTY VS_SHADER_ENTRYPOINT main)
set_property(SOURCE ${PIXELSHADER_FILES} PROPERTY VS_SHADER_MODEL 4.0_level_9_3)
set_property(SOURCE ${PIXELSHADER_FILES} PROPERTY VS_SHADER_FLAGS "/DFLAGS_ADDED /Fh \"$(OutDir)%(Filename).h\"")

set_property(SOURCE ${VERTEXSHADER_FILES} PROPERTY VS_SHADER_TYPE Vertex)
set_property(SOURCE ${VERTEXSHADER_FILES} PROPERTY VS_SHADER_ENTRYPOINT main)
set_property(SOURCE ${VERTEXSHADER_FILES} PROPERTY VS_SHADER_MODEL 4.0_level_9_3)
set_property(SOURCE ${VERTEXSHADER_FILES} PROPERTY VS_SHADER_FLAGS "/DFLAGS_ADDED /Fh \"$(OutDir)%(Filename).h\"")


source_group("Source Files" FILES ${SOURCE_FILES})
source_group("Header Files" FILES ${HEADER_FILES})
source_group("Resource Files" FILES ${RESOURCE_FILES})

add_executable(${EXE_NAME} WIN32 ${SOURCE_FILES} ${HEADER_FILES} ${RESOURCE_FILES})
set_property(TARGET ${EXE_NAME} PROPERTY VS_WINRT_COMPONENT TRUE)
target_link_libraries(${EXE_NAME} d2d1.lib d3d11.lib dxgi.lib dwrite.lib )
