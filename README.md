# Балтаев Тимур ИУ8-22
```sh
После того, как вы настроили взаимодействие с системой непрерывной интеграции,
обеспечив автоматическую сборку и тестирование ваших изменений, стоит задуматься
о создание пакетов для измениний, которые помечаются тэгами (см. вкладку releases).
Пакет должен содержать приложение solver из предыдущего задания Таким образом, каждый новый релиз будет состоять из следующих компонентов:

архивы с файлами исходного кода (.tar.gz, .zip)
пакеты с бинарным файлом solver (.deb, .rpm, .msi, .dmg)
```
```sh
$ git clone https://github.com/BaltaevTimur/lab04
$ touch CPack.cmake
```
## CMakeLists.txt
```sh

cmake_minimum_required(VERSION 3.22)

project(solver)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_library(solver_lib STATIC ${CMAKE_CURRENT_SOURCE_DIR}/solver_lib/solver.cpp)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/solver_lib)
add_library(formatter STATIC ${CMAKE_CURRENT_SOURCE_DIR}/formatter_lib/formatter.cpp)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/formatter_lib)
add_library(formatter_ex STATIC ${CMAKE_CURRENT_SOURCE_DIR}/formatter_ex_lib/formatter_ex.cpp)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/formatter_ex_lib)
add_executable(solver ${CMAKE_CURRENT_SOURCE_DIR}/solver_application/equation.cpp)
target_link_libraries(formatter_ex formatter)
target_link_libraries(solver solver_lib)
target_link_libraries(solver formatter_ex)


install(TARGETS solver
    RUNTIME DESTINATION bin
)

set(CPACK_PACKAGE_VERSION 1.0.0)

include(CPack.cmake)
```
## CPack.cmake
```sh
include(InstallRequiredSystemLibraries)

set(CPACK_PACKAGE_VERSION ${PRINT_VERSION})
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "C++ solve smth")
set(CPACK_RESOURCE_FILE_LICENSE ${CMAKE_CURRENT_SOURCE_DIR}/LICENSE)
set(CPACK_RESOURCE_FILE_README ${CMAKE_CURRENT_SOURCE_DIR}/README.md)

set(CPACK_SOURCE_IGNORE_FILES 
"\\\\.cmake;/build/;/.git/;/.github/"
)

set(CPACK_SOURCE_INSTALLED_DIRECTORIES "${CMAKE_SOURCE_DIR}; /")

set(CPACK_SOURCE_GENERATOR "TGZ;ZIP")

set(CPACK_DEBIAN_PACKAGE_NAME "solverapp-dev")
set(CPACK_DEBIAN_FILE_NAME "solver-${PRINT_VERSION}.deb")
set(CPACK_DEBIAN_PACKAGE_VERSION ${PRINT_VERSION})
set(CPACK_DEBIAN_PACKAGE_ARCHITECTURE "all")
set(CPACK_DEBIAN_PACKAGE_MAINTAINER "BaltaevTimur")
set(CPACK_DEBIAN_PACKAGE_DESCRIPTION "solve smth")
set(CPACK_DEBIAN_PACKAGE_RELEASE 1)

set(CPACK_GENERATOR "DEB")

include(CPack)
```
## Cpack.yml
```sh
name: CPack

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build_packages_Linux:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v2
        with:
          node-version: '20'
      - name: Config solver
        run: cmake ${{github.workspace}} -B ${{github.workspace}}/build -D PRINT_VERSION=1.0.0

      - name: Build solver
        run: cmake --build ${{github.workspace}}/build

      - name: Build package
        run: cmake --build ${{github.workspace}}/build --target package

      - name: Build source package
        run: cmake --build ${{github.workspace}}/build --target package_source

      - name: Make a release
        uses: ncipollo/release-action@v1.10.0
        with:
          tag: v1.0.0
          artifacts: "build/*.deb,build/*.tar.gz,build/*.zip"
          token: ${{ secrets.GITHUB_TOKEN }}
      - name: Artifacts
        uses: actions/upload-artifact@v4
        with:
          name: solver
          path: build/solver

```
## actions.yml
```sh
name: CPack

on:
 push:
  branches: [main]
 pull_request:
  branches: [main]

jobs: 
 build_Linux:

  runs-on: ubuntu-latest

  steps:
  - uses: actions/checkout@v4

  - name: Configure solver
    run: cmake ${{github.workspace}} -B ${{github.workspace}}/build

  - name: Build solver
    run: cmake --build ${{github.workspace}}/build
```
