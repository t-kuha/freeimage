cmake_minimum_required(VERSION 3.10)

project(freeimage LANGUAGES C CXX)
set(CXX_STANDARD 14)

# configs
option(FI_WITH_EXR ".exr support" ON)
option(FI_WITH_JPEG ".jpeg/.jpg support" OFF)
option(FI_WITH_OPENJPEG ".jpeg/.jpg support" OFF)
option(FI_WITH_JXR ".jxr support" OFF)
option(FI_WITH_PNG ".png support" ON)
option(FI_WITH_RAW "RAW support" ON)
option(FI_WITH_TIFF ".tiff/.tif support" OFF)
option(FI_WITH_WEBP ".webp support" ON)

# main library
# aux_source_directory(Source/FreeImage SRC_FI)
# aux_source_directory(Source/FreeImageToolkit SRC_FITK)

set(
    SOURCES 
    # ${SRC_FI}
    Source/FreeImage/BitmapAccess.cpp
    Source/FreeImage/CacheFile.cpp
    Source/FreeImage/ColorLookup.cpp
    Source/FreeImage/Conversion.cpp
    Source/FreeImage/Conversion16_555.cpp
    Source/FreeImage/Conversion16_565.cpp
    Source/FreeImage/Conversion24.cpp
    Source/FreeImage/Conversion32.cpp
    Source/FreeImage/Conversion4.cpp
    Source/FreeImage/Conversion8.cpp
    Source/FreeImage/ConversionFloat.cpp
    Source/FreeImage/ConversionRGB16.cpp
    Source/FreeImage/ConversionRGBA16.cpp
    Source/FreeImage/ConversionRGBAF.cpp
    Source/FreeImage/ConversionRGBF.cpp
    Source/FreeImage/ConversionType.cpp
    Source/FreeImage/ConversionUINT16.cpp
    Source/FreeImage/FreeImage.cpp
    Source/FreeImage/FreeImageIO.cpp
    Source/FreeImage/GetType.cpp
    Source/FreeImage/Halftoning.cpp
    Source/FreeImage/J2KHelper.cpp
    Source/FreeImage/LFPQuantizer.cpp
    Source/FreeImage/MNGHelper.cpp
    Source/FreeImage/MemoryIO.cpp
    Source/FreeImage/MultiPage.cpp
    Source/FreeImage/NNQuantizer.cpp
    Source/FreeImage/PSDParser.cpp
    Source/FreeImage/PixelAccess.cpp
    Source/FreeImage/Plugin.cpp
    Source/FreeImage/PluginBMP.cpp
    Source/FreeImage/PluginCUT.cpp
    Source/FreeImage/PluginDDS.cpp
    Source/FreeImage/PluginG3.cpp
    Source/FreeImage/PluginGIF.cpp
    Source/FreeImage/PluginHDR.cpp
    Source/FreeImage/PluginICO.cpp
    Source/FreeImage/PluginIFF.cpp
    Source/FreeImage/PluginJNG.cpp
    Source/FreeImage/PluginJP2.cpp
    Source/FreeImage/PluginKOALA.cpp
    Source/FreeImage/PluginMNG.cpp
    Source/FreeImage/PluginPCD.cpp
    Source/FreeImage/PluginPCX.cpp
    Source/FreeImage/PluginPFM.cpp
    Source/FreeImage/PluginPICT.cpp
    Source/FreeImage/PluginPNM.cpp
    Source/FreeImage/PluginPSD.cpp
    Source/FreeImage/PluginRAS.cpp
    Source/FreeImage/PluginSGI.cpp
    Source/FreeImage/PluginTARGA.cpp
    Source/FreeImage/PluginWBMP.cpp
    Source/FreeImage/PluginXBM.cpp
    Source/FreeImage/PluginXPM.cpp
    Source/FreeImage/TIFFLogLuv.cpp
    Source/FreeImage/ToneMapping.cpp
    Source/FreeImage/WuQuantizer.cpp
    Source/FreeImage/ZLibInterface.cpp
    Source/FreeImage/tmoColorConvert.cpp
    Source/FreeImage/tmoDrago03.cpp
    Source/FreeImage/tmoFattal02.cpp
    Source/FreeImage/tmoReinhard05.cpp
    # ${SRC_FITK}
    Source/FreeImageToolkit/BSplineRotate.cpp
    Source/FreeImageToolkit/Background.cpp
    Source/FreeImageToolkit/Channels.cpp
    Source/FreeImageToolkit/ClassicRotate.cpp
    Source/FreeImageToolkit/Colors.cpp
    Source/FreeImageToolkit/CopyPaste.cpp
    Source/FreeImageToolkit/Display.cpp
    Source/FreeImageToolkit/Flip.cpp
    Source/FreeImageToolkit/MultigridPoissonSolver.cpp
    Source/FreeImageToolkit/Rescale.cpp
    Source/FreeImageToolkit/Resize.cpp
)

add_library(freeimage ${SOURCES})

# Specify include directories for your library
target_include_directories(
    freeimage PUBLIC
    Source
    third_party/zlib
)
    
# zlib (1.2.11)
add_subdirectory(third_party/zlib)
target_link_libraries(zlibstatic)

# EXR
if (FI_WITH_EXR)
    target_sources(freeimage PRIVATE Source/FreeImage/PluginEXR.cpp)
    # add_subdirectory(third_party/openexr/OpenEXR)
    add_subdirectory(third_party/openexr/IlmBase)
    target_include_directories(
        freeimage PUBLIC 
        third_party/openexr
        # third_party/openexr/IlmBase
        third_party/openexr/IlmBase/Half
        third_party/openexr/IlmBase/Imath
        third_party/openexr/IlmBase/Iex
        ${CMAKE_BINARY_DIR}/third_party/openexr/IlmBase/config
        ${CMAKE_BINARY_DIR}/third_party/openexr/OpenEXR/config
    )

    # generate 
    include(ExternalProject)
    ExternalProject_Add(
        OpenEXR
        SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/third_party/openexr/OpenEXR
        CMAKE_ARGS -S ${CMAKE_CURRENT_SOURCE_DIR}/third_party/openexr/OpenEXR -B ${CMAKE_BINARY_DIR}/third_party/openexr/OpenEXR/
        BUILD_COMMAND "" # disable build & install step
        INSTALL_COMMAND ""
    )
endif(FI_WITH_EXR)

# jpeg
if (FI_WITH_JPEG)
    target_sources(
        freeimage PRIVATE
        Source/FreeImage/PluginJPEG.cpp
        Source/FreeImageToolkit/JPEGTransform.cpp
    )

    include(ExternalProject)
    ExternalProject_Add(
        libjpeg
        SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/third_party/libjpeg
        CONFIGURE_COMMAND ${CMAKE_CURRENT_SOURCE_DIR}/third_party/libjpeg/configure --prefix=${CMAKE_CURRENT_SOURCE_DIR}/Dist
        BUILD_COMMAND ${MAKE}
    )
    target_include_directories(
        freeimage PUBLIC
        third_party/libjpeg
        Source/LibJPEG # tentative
    )
endif (FI_WITH_JPEG)

# OpenJPEG
if (FI_WITH_OPENJPEG)
    target_sources(freeimage PRIVATE Source/FreeImage/PluginJ2K.cpp)
endif(FI_WITH_OPENJPEG)

# JPEG XR
if (FI_WITH_JXR)
    target_sources(freeimage PRIVATE Source/FreeImage/PluginJXR.cpp)
endif(FI_WITH_JXR)

# PNG (1.6.35)
if(FI_WITH_PNG)
    target_sources(freeimage PRIVATE Source/FreeImage/PluginPNG.cpp)
    option(PNG_SHARED "Build static lib" OFF)
    option(PNG_TESTS "Build libpng tests" OFF)
    option(PNG_DEBUG "Build zlib as static lib" OFF)
    set(CMAKE_CXX_FLAGS -fPIC)
    set(CMAKE_C_FLAGS -fPIC)
    add_subdirectory(third_party/libpng)
    target_include_directories(freeimage PUBLIC third_party/libpng)
    target_include_directories(freeimage PUBLIC ${CMAKE_BINARY_DIR}/third_party/libpng)
    target_link_libraries(freeimage PUBLIC png_static)
endif(FI_WITH_PNG)    

# RAW
if (FI_WITH_RAW)
    target_sources(freeimage PRIVATE Source/FreeImage/PluginRAW.cpp)
endif(FI_WITH_RAW)

# TIFF
if (FI_WITH_TIFF)
    target_sources(freeimage PRIVATE Source/FreeImage/PluginTIFF.cpp)
endif(FI_WITH_TIFF)

# WebP
if (FI_WITH_WEBP)
    add_subdirectory(third_party/libwebp)
    target_include_directories(freeimage PUBLIC third_party/libwebp)
    target_link_libraries(webp)
endif(FI_WITH_WEBP)    


# install headers and the library
install(FILES Source/FreeImage.h DESTINATION ${CMAKE_INSTALL_PREFIX}/include)
install(TARGETS freeimage DESTINATION ${CMAKE_INSTALL_PREFIX}/lib)
