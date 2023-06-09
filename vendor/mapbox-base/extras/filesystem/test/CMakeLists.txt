
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_CURRENT_SOURCE_DIR}/cmake/")
set(PARSE_CATCH_TESTS_ADD_TO_CONFIGURE_DEPENDS ON)
include(ParseAndAddCatchTests)

if(GHC_COVERAGE)
    message("Generating test runner for coverage run...")
    set(CMAKE_EXE_LINKER_FLAGS "${CMCMAKE_EXE_LINKER_FLAGS} --coverage") 
    add_executable(filesystem_test filesystem_test.cpp catch.hpp)
    if(MINGW)
        target_compile_options(filesystem_test PUBLIC --coverage "-Wa,-mbig-obj")
    else()
        target_compile_options(filesystem_test PUBLIC --coverage)
    endif()
    target_link_libraries(filesystem_test PUBLIC ghc_filesystem --coverage)
else()
    message("Generating test runner for normal test...")
    add_executable(filesystem_test filesystem_test.cpp catch.hpp)
    target_link_libraries(filesystem_test ghc_filesystem)
    target_compile_options(filesystem_test PRIVATE
        $<$<CXX_COMPILER_ID:Clang>:-Wall -Wextra -Wshadow -Werror>
        $<$<CXX_COMPILER_ID:GNU>:-Wall -Wextra -Wshadow -Werror>
        $<$<CXX_COMPILER_ID:MSVC>:/WX>)
    if(CMAKE_CXX_COMPILER_ID MATCHES MSVC)
        target_compile_definitions(filesystem_test PRIVATE _CRT_SECURE_NO_WARNINGS)
    endif()
    ParseAndAddCatchTests(filesystem_test)
    AddExecutableWithStdFS(std_filesystem_test filesystem_test.cpp catch.hpp)
    if(WIN32)
        add_executable(filesystem_test_wchar filesystem_test.cpp catch.hpp)
        target_link_libraries(filesystem_test_wchar ghc_filesystem)
        target_compile_options(filesystem_test_wchar PRIVATE
            $<$<CXX_COMPILER_ID:Clang>:-Wall -Wextra -Werror>
            $<$<CXX_COMPILER_ID:GNU>:-Wall -Werror>
            $<$<CXX_COMPILER_ID:MSVC>:/WX>)
        if(CMAKE_CXX_COMPILER_ID MATCHES MSVC)
            target_compile_definitions(filesystem_test_wchar PRIVATE _CRT_SECURE_NO_WARNINGS GHC_WIN_WSTRING_STRING_TYPE)
        else()
            target_compile_definitions(filesystem_test_wchar PRIVATE GHC_WIN_WSTRING_STRING_TYPE)
        endif()
        ParseAndAddCatchTests(filesystem_test_wchar)
    endif()
endif()

add_executable(multifile_test multi1.cpp multi2.cpp catch.hpp)
target_link_libraries(multifile_test ghc_filesystem)
add_test(multifile_test multifile_test)

add_executable(fwd_impl_test fwd_test.cpp impl_test.cpp)
target_link_libraries(fwd_impl_test ghc_filesystem)
if(CMAKE_CXX_COMPILER_ID MATCHES MSVC)
    target_compile_definitions(fwd_impl_test PRIVATE _CRT_SECURE_NO_WARNINGS)
endif()
add_test(fwd_impl_test fwd_impl_test)
