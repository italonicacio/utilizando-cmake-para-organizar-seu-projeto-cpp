# [CONTEÚDO] Utilizando CMake para organizar seu projeto C++

## E o que é CMake?
Quem já usou [Make](https://www.gnu.org/software/make/manual/make.html), conhecido também por seus arquivos makefile, sabe que estruturar o sistema de compilação do projeto pode ser algo bem demorado. O [CMake](https://cmake.org) vem para ajudar nesse sentido, ele é um software multiplataforma de geração automatizada, no nosso caso ele gera os arquivos do Make, no [site oficial](https://cmake.org/features/) diz também que ele pode gerar os arquivos scripts de outros softwares de compilação como o [Ninja Build](https://ninja-build.org). 

## O que vai ser abordado aqui?

- CMake para geração dos arquivos de build.
- Build com arquivos makefile.
- Baixar e disponibilizar o [Google Test](https://github.com/google/googletest) (gtest) como framework de testes.

## Projeto exemplo

Vamos dizer que estamos criando uma biblioteca, que vai ser usada em outro projeto, nessa biblioteca temos algumas operações (como exemplo temos as operações de soma e subtração) e também uma das coisas mais importantes no desenvolvimento, a parte de testes. 

O nosso projeto de exemplo está estruturado da seguinte forma:

```shell
my-lib
.
├── lib
│   ├── include
│   │   ├── subtraction.h
│   │   └── sum.h
│   └── src
│       ├── subtraction.cpp
│       └── sum.cpp
├── README.md
└── tests
    └── unit-tests
        ├── subtraction-test.cpp
        ├── sum-test.cpp
        └── tests.cpp
```

## Arquivos CMake

Primeiro a raiz do nosso projeto vamos incluir um arquivo CMake chamado *CMakeLists.txt*. Nos arquivos *CMakeLists* é onde definimos toda estrutura dos nossos projetos.

```cmake
cmake_minimum_required(VERSION 3.22)

set(CMAKE_CXX_STANDARD 20)

project(my-project VERSION 1.0)

add_subdirectory(lib)

include(FetchContent)
add_subdirectory(tests/unit-tests)
```

Com o **cmake_minimum_required** conseguimos definir a versão minima CMake para executar esse *CMakeLists.txt*, em algum projeto talvez seja necessário utilizar por exemplo o **2.8...3.22**  em vez de apenas **3.22** para definir a versão, essa primeira forma defini uma versão de compatibilidade com a versão *2.8*, porem indica que utilizamos a versão *3.22*. A variável CMAKE_CXX_STANDARD é utilizada para definir uma versão minima do C++, nesse projeto vamos utilizar a o [C++20](https://en.cppreference.com/w/cpp/20).

O **project** definimos o nome do projeto e a versão que estamos desenvolvendo. Por último fazemos a adição dos dois subdiretórios que é a pasta lib e a tests. Essa adição de subdiretórios faz com que o CMake procure o arquivo *CMakeLists.txt* dentro das pastas lib e tests. O FetchContent vai ser incluído aqui para podermos baixar o *Google Test* e disponibilizar ele para nossos testes.

### CmakeLists da pasta lib

O *CMakeLists* da pasta *lib* vai ficar da seguinte forma:

```cmake
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
```

O **add_library** define o nome da biblioteca e os arquivos do código fonte, o **SHARED** define o tipo de biblioteca a ser criada, junto do *SHARED* temos também o *STATIC* e o *MODULE*, os arquivos *.cpp* podem ser definidos em uma lista chamada *SOURCE_DIRS* e passando para o *add_library*, vai ao seu critério como você quer montar o projeto.

O **target_include_directories** aponta onde podemos encontrar os diretórios do projeto, se existe mais de uma pasta com os arquivos cabeçalhos (*.h* ou *.hpp*), você deve colocar o caminho até essas pastas nesta função, pode se criar também uma variável chamada *INCLUDE_DIRS* para definir os caminhos até os arquivos.

E finalmente o **install**, nele você seleciona a biblioteca a ser criada, com o *TARGETS* você define qual a biblioteca e em seguida coloca onde vai ser criada a biblioteca, no nosso caso eu escolhi a pasta padrão que o CMake cria, que é a pasta lib. Também é possível selecionar onde vai ser posto os arquivos cabeçalhos, no nosso caso usamos a variável *CMAKE_INSTALL_INCLUDEDIR* para direcionar os arquivos de cabeçalho da nossa biblioteca. 

Existem outras variáveis para configurar melhor seu projeto, geralmente variáveis do CMake iniciam com o *CMAKE_*.

### CmakeLists da pasta tests

E na pasta *tests* o nosso ultimo arquivo *CMakeLists*.

```cmake
set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)

FetchContent_Declare(
    googletest
    GIT_REPOSITORY https://github.com/google/googletest.git
    GIT_TAG        release-1.12.1
)

FetchContent_MakeAvailable(googletest)

add_executable(
    LibUnitTests
    tests.cpp
    sum-test.cpp
    subtraction-test.cpp
)

target_include_directories(LibUnitTests PRIVATE ${CMAKE_SOURCE_DIR}/lib/include/)

target_link_libraries(LibUnitTests PRIVATE gtest_main my-lib)
```

A primeira linha, temos uma configuração para o gtest, diz basicamente que quer compilar o gtest de forma compartilhada isso evita alguns erros de linkagem de bibliotecas, para nosso caso é bastante utilizar isso, mas se quiserem ler mais sobre [cliquem aqui](https://github.com/google/googletest/blob/main/googletest/README.md#visual-studio-dynamic-vs-static-runtimes).

O **FetchContent** é utilizado para baixar projetos externos, aqui utilizamos para baixar o gtest com o **FetchContent_Declare**, o primeiro parâmetro é o nome do projeto externo que você quer utilizar, com o **GIT_REPOSITORY** dizemos que é uma repositório e colocamos o link apontando para o repositório, o **GIT_TAG** conseguimos selecionar qual versão queremos baixar e utilizar. Em seguida temos **FetchContent_MakeAvailable** que indica que queremos deixar o gtest pronto para uso na nossa aplicação, ou seja, ele vai compilar o projeto e disponibilizar para a aplicação.

O **target_include_directories** é conhecido já, definimos um caminho com o **${CMAKE_SOURCE_DIR}/lib/include/** para os testes encontrarem os arquivos cabeçalhos da biblioteca, e o valor da variável *CMAKE_SOURCE_DIR* é o caminho até a raiz do projeto.

> Obs.: Para entender o níveis de acesso *PRIVATE* e *PUBLIC*, [clique aqui](https://cmake.org/cmake/help/latest/command/target_include_directories.html) para ler a documentação detalhada dessa função do CMake.

Por ultimo temos a **target_link_libraries** que tem uma função semelhante a *target_include_directories*, só que ela é utilizada para definir quais bibliotecas são necessárias para a linkagem correta do programa.

### Nova estrutura de arquivos e pasta

Com a adição desses novos *CMakeLists* temos a seguinte estrutura de arquivos e diretórios.

```shell
my-lib
.
├── CMakeLists.txt
├── lib
│   ├── CMakeLists.txt
│   ├── include
│   │   ├── subtraction.h
│   │   └── sum.h
│   └── src
│       ├── subtraction.cpp
│       └── sum.cpp
├── README.md
└── tests
    └── unit-tests
        ├── CMakeLists.txt
        ├── subtraction-test.cpp
        ├── sum-test.cpp
        └── tests.cpp
```

## Executando o projeto

### Gerando arquivos Make (Makefile)

Bom, agora podemos por fim gerar os arquivos Make e compilar nosso projeto. Primeiro vamos criar uma pasta chamada build e entrar nela, em distros Linux podemos fazer isso com o comando abaixo.

```shell
 mkdir build && cd build 
```
Dentro da pasta build, agora basta executar o comando para ser feito a geração dos arquivos makefile.

```shell
 cmake ../
```

Se durante a compilação do projeto você quer ver o que está sendo feito, basta você pode passar uma variável de configuração para o CMake.

```shell
 cmake ../ -DCMAKE_VERBOSE_MAKEFILE=ON
```
Essa variável explicita que os arquivos makefile sejam verboso e mostrem todo o processo de compilação.

Após executar esse comando você deve obter algo parecido com a imagem abaixo.

![cmake-exemplo](https://drive.google.com/uc?export=view&id=1wEJGi-ZTo5_mbOcUemaLpkPOYHQmxKZI)


### Compilando o projeto

Para compilar todo o projeto, fazer o download do gtest, criar a biblioteca e compilar os códigos de teste basta executar o seguinte comando.

```shell
 make
```

![make-exemple](https://drive.google.com/uc?export=view&id=14N4laUnXYKAGOcwkbKbzqG5DuBO6dX-b)

### Tá, mas eae? Funcionou?

Bom funcionou, mas para que você acredite em mim vamos até a pasta lib dentro da pasta build.

![ls-exemplo-lib](https://drive.google.com/uc?export=view&id=1ppP6PAKR1R2-ZeBq9mDvtH-dhPuaEBOe)


E também podemos executar os testes unitários.

![exemplo-testes-unitarios](https://drive.google.com/uc?export=view&id=1j8efZuWzLJsst14l12231djqUPJMBSWb)


## Mais algumas coisas do CMake

O CMake tem um grande ferramental para lidar com essa parte de organização, cada um tem liberdade de arquitetar seu projeta da melhor forma e com o CMake isso é possível. 

Abaixo temos mais algumas coisas do CMake.

- [set_target_properties](https://cmake.org/cmake/help/latest/command/set_target_properties.html) - Se eu não estivesse fazendo uma biblioteca, estou fazendo um programa CLI que faz a correção de arquivos texto em batchs. Ainda é possível disponibilizar meu código para os testes por exemplo, sem necessariamente ficar recompilando o projeto? Sim, com o *set_target_properties* com as seguintes propriedades *ENABLE_EXPORTS ON*
e *POSITION_INDEPENDENT_CODE ON*.
- [find_package](https://cmake.org/cmake/help/latest/command/find_package.html) - O *find_package*  é usado para procurar no seu computador se a biblioteca que você pediu já está instalada.

### Condicionais 

Se eu quiser fazer os testes unitários apenas se eu passar uma flag para o CMake? Como posso fazer? 

Bom essa é simples de responder, você pode fazer da seguinte forma. 

```cmake
if(testing_enabled) 
    add_subdirectory(tests/unit-tests)
endif()
```
E no terminal quando executar o comando do CMake, basta colocar *-Dtesting_enabled=True* como um parâmetro a mais e pronto. Se no arquivo CMakeList onde você colocou esse if tiver a variável definida para True, não é necessário passar, mas se não definir a variável, por padrão vai dar falso no teste condicional do if.

```bash
cmake ../ -Dtesting_enabled=True 
```
--------
**Repositório com o código:** <https://github.com/italonicacio/utilizando-cmake-para-organizar-seu-projeto-cpp/tree/main/project>