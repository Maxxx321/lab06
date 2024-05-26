После того, как вы настроили взаимодействие с системой непрерывной интеграции,
обеспечив автоматическую сборку и тестирование ваших изменений, стоит задуматься
о создание пакетов для измениний, которые помечаются тэгами (см. вкладку releases).
Пакет должен содержать приложение solver из предыдущего задания Таким образом, каждый новый релиз будет состоять из следующих компонентов:

архивы с файлами исходного кода (.tar.gz, .zip)
пакеты с бинарным файлом solver (.deb, .rpm, .msi, .dmg)

Для этого нужно добавить ветвление в конфигурационные файлы для CI со следующей логикой:
если commit помечен тэгом, то необходимо собрать пакеты (DEB, RPM, WIX, DragNDrop, ...)
и разместить их на сервисе GitHub. (см. пример для Travi CI)

1. Клонируем репозиторий с 4 лабораторной

`````sh
git clone https://github.com/${GITHUB_USERNAME}/lab04 lab06
cd lab06
git remote remove origin
git remote add origin https://github.com/${GITHUB_USERNAME}/lab06
`````

`````sh
Cloning into 'lab06'...
remote: Enumerating objects: 327, done.
remote: Counting objects: 100% (327/327), done.
remote: Compressing objects: 100% (153/153), done.
remote: Total 327 (delta 145), reused 286 (delta 128), pack-reused 0
Receiving objects: 100% (327/327), 1.12 MiB | 5.27 MiB/s, done.
Resolving deltas: 100% (145/145), done.
`````

2. Cmake в lab06

`````sh
	cat > CMakeLists.txt <<EOF
	cmake_minimum_required(VERSION 3.4)
	project(lab06)
	set(CMAKE_CXX_STANDARD 11)
	set(CMAKE_CXX_STANDARD_REQUIRED ON)

	add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/solver_application)

	include(CPack.cmake)
	EOF
`````

3. Cpack

`````sh
	cat > CPack.cmake <<EOF
	include(InstallRequiredSystemLibraries)
	
	set(CPACK_PACKAGE_CONTACT eisnersandy2@gmail.com)
	set(CPACK_PACKAGE_VERSION ${PRINT_VERSION})
	set(CPACK_PACKAGE_NAME "solver")
	set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "C++ library")
	set(CPACK_PACKAGE_PACK_NAME "solver-${PRINT_VERSION}")
	
	set(CPACK_SOURCE_INSTALLED_DIRECTORIES 
	  "${CMAKE_SOURCE_DIR}/solver_application; solver_application"
	  "${CMAKE_SOURCE_DIR}/solver_lib; solver_lib"
	  "${CMAKE_SOURCE_DIR}/formatter_ex_lib; formatter_ex_lib"
	  "${CMAKE_SOURCE_DIR}/formatter_lib; formatter_lib")
	
	set(CPACK_RESOURCE_FILE_README ${CMAKE_CURRENT_SOURCE_DIR}/README.md)
	
	set(CPACK_SOURCE_GENERATOR "TGZ;ZIP")
	
	set(CPACK_DEBIAN_PACKAGE_PREDEPENDS "cmake >= 3.0")
	set(CPACK_DEBIAN_PACKAGE_RELEASE 1)
	
	set(CPACK_DEBIAN_PACKAGE_VERSION ${PRINT_VERSION})
	set(CPACK_DEBIAN_PACKAGE_ARCHITECTURE "all")
	set(CPACK_DEBIAN_PACKAGE_DESCRIPTION "it's solving")
	
	set(CPACK_GENERATOR "DEB;RPM")
	
	set(CPACK_RPM_PACKAGE_SUMMARY "solves equations")
	
	include(CPack)
	EOF
	
	git push origin main
`````

`````sh
Enumerating objects: 327, done.
Counting objects: 100% (327/327), done.
Delta compression using up to 8 threads
Compressing objects: 100% (136/136), done.
Writing objects: 100% (327/327), 1.12 MiB | 1.55 MiB/s, done.
Total 327 (delta 145), reused 327 (delta 145), pack-reused 0
remote: Resolving deltas: 100% (145/145), done.
To https://github.com/Titanoboba/lab06
 * [new branch]      main -> main
`````

4. Workflow

Файл - для выполнения build.

`````sh
name: CMake

on:
 push:
  branches: [main]
 pull_request:
  branches: [main]

jobs: 
 build_Linux:

  runs-on: ubuntu-latest

  steps:
  - uses: actions/checkout@v3

  - name: Configure Solver
    run: cmake ${{github.workspace}} -B ${{github.workspace}}/build

  - name: Build Solver
    run: cmake --build ${{github.workspace}}/build
`````

Файл - build and release

`````sh
name: CMake Build and Release

on:
  push:
    tags:
      - 'v*.*.*'

jobs:
  build_and_release:
    runs-on: ubuntu-latest
    permissions:
      contents: write

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v3

      - name: Configure Solver
        run: cmake ${{github.workspace}} -B ${{github.workspace}}/build -D PRINT_VERSION=${GITHUB_REF_NAME#v}

      - name: Build Solver
        run: cmake --build ${{github.workspace}}/build

      - name: Build Package
        run: cmake --build ${{github.workspace}}/build --target package

      - name: Build Source Package
        run: cmake --build ${{github.workspace}}/build --target package_source

      - name: Make a Release
        uses: ncipollo/release-action@v1.14.0
        with:
          artifacts: |
            DEB
            RPM
            tar.gz
            zip
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
`````