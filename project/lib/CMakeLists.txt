
add_library(
    my-lib 
        SHARED
            src/sum.cpp
            src/subtraction.cpp
)

target_include_directories(my-lib PUBLIC include)

install(
    TARGETS my-lib
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)