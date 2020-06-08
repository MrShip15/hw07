[![Build Status](https://travis-ci.org/MrShip15/hw07.svg?branch=master)](https://travis-ci.org/MrShip15/hw07)
[![Build status](https://ci.appveyor.com/api/projects/status/mx2hplisib85u4k2?svg=true)](https://ci.appveyor.com/project/MrShip15/hw07)
## Homework VII

### Задание
Создадим настройки менеджера пакетов **Hunter** для репозитория с ДЗ № 5
Настройка git-репозитория **hw07** для работы
```sh
% git clone https://github.com/MrShip15/hw05 hw07
% cd hw07
% git remote remove origin
% git remote add origin https://github.com/MrShip15/hw07
```
Скачивание и подключение модуля `HunterGate`
```sh
% mkdir -p cmake # Создание директории где будут храниться файлы Hunter
# Скачивание данных из файла в удаленном репозитории и их запись в файл HunterGate.cmake
% wget https://raw.githubusercontent.com/cpp-pm/gate/master/cmake/HunterGate.cmake -O cmake/HunterGate.cmake
# Добавление HunterGate к CMake
% gsed -i "" '/cmake_minimum_required(VERSION 3.10)/a\

include("cmake/HunterGate.cmake")
HunterGate(
    URL "https:\//github.com/cpp-pm/hunter/archive/v0.23.251.tar.gz"
    SHA1 "5659b15dc0884d4b03dbd95710e6a1fa0fc3258d"
)
' CMakeLists.txt
```
Теперь не нужно скачивать **GTest** самостоятельно. **Hunter** сам подтянет добавленные с помощью функции `hunter_add_package`.
```sh
# Удаление подмодуля с GTest
% git rm -rf third-party/gtest
# Добавление через hunter пакета gtest и его поиск
% gsed -i "" '/option(BUILD_TESTS "Build tests" OFF)/a\

hunter_add_package(GTest)
find_package(GTest CONFIG REQUIRED)
' CMakeLists.txt
# Удаление строки с добавлением поддиректории gtest
% gsed -i "" 's/add_subdirectory(third-party/gtest)//' CMakeLists.txt
# Замена обращение к gtest gtest_main на GTest::gtest_main
% gsed -i "" 's/gtest_main/GTest::gtest_main/' CMakeLists.txt
```
Сборка прокта при помощи **Hunter**.
```sh
# Видим как полключаются пакеты при помощи Hanter'a
% cmake -H. -B_builds -DBUILD_TESTS=ON
% cmake --build _builds
% cmake --build _builds --target test
```
Добавление конфигурационного файла в проект, который будет содержать необходимую версию GTest.
```sh
% mkdir cmake/Hunter
# Установка нужной версии GTest
% cat > cmake/Hunter/config.cmake <<EOF
hunter_config(GTest VERSION 1.7.0-hunter-9)
EOF
# add LOCAL in HunterGate function
```
Добавление подмодуля **polly**, который содержит инструкции для сборки проектов с установленным **Hunter**.
```sh
% mkdir tools
% git submodule add https://github.com/ruslo/polly tools/polly
% tools/polly/bin/polly.py --test
# Сборка проекта на компиляторе со станартом с++14
% tools/polly/bin/polly.py --toolchain clang-cxx14
```
Добавим непрерывную интеграцию с **Appveyor**
Создание `appveyor.yml`
```sh
% cat >> appveyor.yml <<EOF
image: Visual Studio 2019
platform:
  - x86
  - x64
configuration: Release

build_script:
  - cmd: cmake -H. -B_build -DBUILD_TESTS=ON
  - cmd: cmake --build _build
EOF
```
## Links

- [Create Hunter package](https://docs.hunter.sh/en/latest/creating-new/create.html)
- [Custom Hunter config](https://github.com/ruslo/hunter/wiki/example.custom.config.id)
- [Polly](https://github.com/ruslo/polly)

```
Copyright (c) 2015-2020 The ISC Authors
```
