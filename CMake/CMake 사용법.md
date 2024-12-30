
CMakeLists.txt를 만들자

로그가 의미하는 것?

Configuring done - 


Configure 단계

1. CMakeListss.txt 분석
2. CMakeCache.txt 생성

Generating done - 

Generate 단계
1.  Native Build System 파일 생성

주의점: CMakeCache.txt에 정의된 변수를 사용하기 때문에 갱신되지 않으면 잘못된 Native Build Systme file이 생성될 수 있음.


--fresh를 사용하면 CMakeCache.txt를 지우고 빌드함


----
CMake 커맨드


cmake_minimum_required(VERSION <version>)


project(<project-name> VERSION <version> LANGUAGES <language-name>)


message(""), message(STATUS "")



set(<variable-name> <variable-value>)
message("${variable-name}")
unset(<variable-name>)


List(<sub-command> <list> ...) 보통 APPEND와 FIND를 주로 사용
예시:
List(APPEND FILES foo.cpp bar.cpp haz.cpp)
List(FIND FILES bar.cpp bar_cpp_index)
List<SORT FILES> // 알파벳 순서도로 정렬

if(<condition>)
elseif(<condition>)
else()
endif()

foreach(<loop_var> IN [LISTS <lists> ] [ ITEMS <items>]
endforeach()
예시:
foreach(FILE IN LISTS FILES)
	message("${FILE} " is in the list)
endforeach()

add_subdirectory(<source_directory>)
폴더마다 CMakeLists.txt를 가질 수 있음
CMake 실행할 때 지정된 CMakeLists.txt가 실행됨 (하위폴더는 포함되지 않음)
add_subdirectory를 통해서 하위 폴더 추가 가능 (소스 디렉토리에 CMakeLists.txt 필수)


add_compile_options(<option>)

컴파일 옵션을 전역적으로 추가