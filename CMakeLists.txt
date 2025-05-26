cmake_minimum_required(VERSION 3.16)
project(ImGuiDirectX11Example)

# 設定 C++ 標準
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# 設定輸出目錄
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)

# 包含 FetchContent 模組
include(FetchContent)

# 使用 FetchContent 從 GitHub 自動下載 ImGui（固定版本）
message(STATUS "正在下載 ImGui...")

FetchContent_Declare(
    imgui
    GIT_REPOSITORY https://github.com/ocornut/imgui.git
    GIT_TAG v1.90.4
)

FetchContent_MakeAvailable(imgui)

# 設定 ImGui 目錄變數
set(IMGUI_DIR ${imgui_SOURCE_DIR})
message(STATUS "使用 ImGui 版本來自: ${IMGUI_DIR}")

# ImGui 核心與後端檔案
set(IMGUI_SOURCES
    ${IMGUI_DIR}/imgui.cpp
    ${IMGUI_DIR}/imgui_demo.cpp
    ${IMGUI_DIR}/imgui_draw.cpp
    ${IMGUI_DIR}/imgui_tables.cpp
    ${IMGUI_DIR}/imgui_widgets.cpp
)

set(IMGUI_BACKENDS
    ${IMGUI_DIR}/backends/imgui_impl_win32.cpp
    ${IMGUI_DIR}/backends/imgui_impl_dx11.cpp
)

set(IMGUI_INCLUDE_DIRS
    ${IMGUI_DIR}
    ${IMGUI_DIR}/backends
)

# 你的專案原始碼
set(PROJECT_SOURCES
    src/main.cpp
    # 可以在此添加其他檔案
)

# 創建執行檔
add_executable(${PROJECT_NAME}
    ${PROJECT_SOURCES}
    ${IMGUI_SOURCES}
    ${IMGUI_BACKENDS}
)



add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E echo "Ensuring output directory exists..."
    COMMAND ${CMAKE_COMMAND} -E make_directory "$<SHELL_PATH:${CMAKE_CURRENT_SOURCE_DIR}/output>"
    COMMAND ${CMAKE_COMMAND} -E echo "Copying executable to output/"
    COMMAND ${CMAKE_COMMAND} -E copy_if_different
        "$<TARGET_FILE:${PROJECT_NAME}>"
        "$<SHELL_PATH:${CMAKE_CURRENT_SOURCE_DIR}/output/>"
    COMMENT "Copying build result to output/ folder..."
)




# 設定 include 目錄
target_include_directories(${PROJECT_NAME} PRIVATE
    ${IMGUI_INCLUDE_DIRS}
    src/
)

# Windows 專用設定
if(WIN32)
    target_link_libraries(${PROJECT_NAME}
        d3d11
        dxgi
        d3dcompiler
        winmm
        imm32
        ole32
        oleaut32
        uuid
        kernel32
        user32
        gdi32
        winspool
        shell32
        comdlg32
        advapi32
    )

    set_target_properties(${PROJECT_NAME} PROPERTIES
        WIN32_EXECUTABLE TRUE
        VS_DEBUGGER_WORKING_DIRECTORY "${CMAKE_RUNTIME_OUTPUT_DIRECTORY}"
    )

    # 如果你想開啟 console 除錯視窗
    # set_target_properties(${PROJECT_NAME} PROPERTIES LINK_FLAGS "/SUBSYSTEM:CONSOLE")
endif()

# 編譯器設定
if(MSVC)
    target_compile_definitions(${PROJECT_NAME} PRIVATE
        _CRT_SECURE_NO_WARNINGS
        NOMINMAX
        WIN32_LEAN_AND_MEAN
    )
    target_compile_options(${PROJECT_NAME} PRIVATE /W3 /MP)
elseif(CMAKE_CXX_COMPILER_ID STREQUAL "GNU" OR CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
    target_compile_options(${PROJECT_NAME} PRIVATE -Wall -Wextra)
endif()

# Debug/Release 設定
if(CMAKE_BUILD_TYPE STREQUAL "Debug")
    target_compile_definitions(${PROJECT_NAME} PRIVATE DEBUG _DEBUG)
else()
    target_compile_definitions(${PROJECT_NAME} PRIVATE NDEBUG)
endif()

# 安裝目標（可選）
install(TARGETS ${PROJECT_NAME}
    RUNTIME DESTINATION bin
)

# 顯示設定資訊
message(STATUS "")
message(STATUS "==========================================")
message(STATUS "  ✅ ImGui DirectX11 專案設定完成")
message(STATUS "  建構類型: ${CMAKE_BUILD_TYPE}")
message(STATUS "  C++ 標準: ${CMAKE_CXX_STANDARD}")
message(STATUS "  ImGui 來源: ${IMGUI_DIR}")
message(STATUS "  輸出目錄: ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}")
message(STATUS "==========================================")
message(STATUS "")
