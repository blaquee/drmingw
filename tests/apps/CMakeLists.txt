cmake_minimum_required (VERSION 2.6)

project (drmingw-test-apps)

include (CheckCSourceCompiles)

unset (CMAKE_RUNTIME_OUTPUT_DIRECTORY)
unset (CMAKE_LIBRARY_OUTPUT_DIRECTORY)

if (MINGW)
    check_c_source_compiles ("#include <windows.h>\nstruct _EXCEPTION_REGISTRATION_RECORD r; int main() { return 0; }" HAVE_EXCEPTION_REGISTRATION_RECORD)
    if (HAVE_EXCEPTION_REGISTRATION_RECORD)
        add_definitions (-DHAVE_EXCEPTION_REGISTRATION_RECORD=1)
    endif ()

    # Match MSVC default stack size of 1MB
    # https://msdn.microsoft.com/en-us/library/8cxs58a6.aspx
    set (CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -Wl,--stack,1048576")
endif ()

if (MSVC)
    if (CMAKE_VS_WINDOWS_TARGET_PLATFORM_VERSION)
        message (STATUS "Using Windows SDK ${CMAKE_VS_WINDOWS_TARGET_PLATFORM_VERSION}")
    endif ()

    # http://www.cmake.org/Wiki/CMake_FAQ#How_can_I_build_my_MSVC_application_with_a_static_runtime.3F
    foreach (flag_var
        CMAKE_C_FLAGS CMAKE_C_FLAGS_DEBUG CMAKE_C_FLAGS_RELEASE CMAKE_C_FLAGS_MINSIZEREL CMAKE_C_FLAGS_RELWITHDEBINFO
        CMAKE_CXX_FLAGS CMAKE_CXX_FLAGS_DEBUG CMAKE_CXX_FLAGS_RELEASE CMAKE_CXX_FLAGS_MINSIZEREL CMAKE_CXX_FLAGS_RELWITHDEBINFO
    )
        if (${flag_var} MATCHES "/MD")
            string (REGEX REPLACE "/MD" "/MT" ${flag_var} "${${flag_var}}")
        endif ()
    endforeach ()

    # Allows expansion only of functions marked inline, __inline, or
    # __forceinline, or in a C++ member function defined in a class
    # declaration.
    string (REGEX REPLACE "/Ob0" "/Ob1" CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG}")
    string (REGEX REPLACE "/Ob0" "/Ob1" CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG}")

    add_definitions (-D_CRT_SECURE_NO_DEPRECATE -D_CRT_SECURE_NO_WARNINGS)
    add_definitions (-D_SCL_SECURE_NO_DEPRECATE -D_SCL_SECURE_NO_WARNINGS)
endif ()

add_definitions (-DEXIT_SKIP=125)

macro (add_test_executable)
    add_executable (${ARGV})
    if (NOT ${CMAKE_SOURCE_DIR} STREQUAL ${CMAKE_CURRENT_SOURCE_DIR})
        add_dependencies (check ${ARGV0})
    endif ()
endmacro ()

add_test_executable (_exit3 _exit3.c)
add_test_executable (abort_console abort_console.c)
add_test_executable (abort_gui WIN32 abort_gui.c)
add_test_executable (access_violation access_violation.c)
add_test_executable (access_violation_cxx access_violation_cxx.cpp)
add_test_executable (assert_console assert_console.c)
add_test_executable (assert_gui WIN32 assert_gui.c)
add_test_executable (ctrl_break ctrl_break.c)
add_test_executable (ctrl_c ctrl_c.c)
add_test_executable (crt_dbg_report_assert_debug crt_dbg_report_assert_debug.c)
add_test_executable (crt_dbg_report_assert_stderr crt_dbg_report_assert_stderr.c)
add_test_executable (crt_dbg_report_assert_wndw crt_dbg_report_assert_wndw.c)
add_test_executable (cxx_exception_handled cxx_exception_handled.cpp)
add_test_executable (cxx_exception_unhandled cxx_exception_unhandled.cpp)
add_test_executable (cxx_inline cxx_inline.cpp)
add_test_executable (debug_break debug_break.c)
add_test_executable (dialog_box WIN32 dialog_box.c dialog_box_rc.rc)
add_test_executable (false false.c)
add_test_executable (fast_fail fast_fail.c)
add_test_executable (infinite_loop infinite_loop.c)
add_test_executable (int3 int3.c)
add_test_executable (int_divide_by_zero int_divide_by_zero.c)
add_test_executable (int_overflow int_overflow.c)
add_test_executable (invalid_parameter invalid_parameter.c)
add_test_executable (invoke_watson invoke_watson.c)
add_test_executable (is_debugger_present is_debugger_present.c)
add_test_executable (message_box WIN32 message_box.c)
add_test_executable (nt_assert nt_assert.c)
add_test_executable (output_debug_string_a WIN32 output_debug_string_a.c)
add_test_executable (output_debug_string_w WIN32 output_debug_string_w.c)
add_test_executable (seh_handled seh_handled.c)
add_test_executable (seh_unhandled WIN32 seh_unhandled.c)
add_test_executable (set_thread_name set_thread_name.c)
add_test_executable (stack_overflow stack_overflow.c)
add_test_executable (std_terminate std_terminate.cpp)
add_test_executable (true true.c)
add_test_executable (ud2 ud2.c)
