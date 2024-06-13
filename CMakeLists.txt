cmake_minimum_required(VERSION 3.7)

if (CMAKE_VERSION VERSION_GREATER_EQUAL "3.24.0")
	cmake_policy(SET CMP0135 NEW)
endif()

project(macphp LANGUAGES C)

include(ExternalProject)

execute_process(COMMAND xcrun --show-sdk-path OUTPUT_VARIABLE MACOS_SDK_PREFIX OUTPUT_STRIP_TRAILING_WHITESPACE)

ExternalProject_Add(ext_openssl
  URL "https://github.com/openssl/openssl/releases/download/openssl-3.3.1/openssl-3.3.1.tar.gz"
  URL_HASH SHA256=777cd596284c883375a2a7a11bf5d2786fc5413255efab20c50d6ffe6d020b7e
  PREFIX build/deps/openssl
  INSTALL_DIR ${CMAKE_INSTALL_PREFIX}
  BUILD_IN_SOURCE 1
  CONFIGURE_COMMAND ./Configure --prefix=${CMAKE_INSTALL_PREFIX} --openssldir=${CMAKE_INSTALL_PREFIX}/ssl ${OPENSSL_PLATFORM} no-tests no-asm shared
  BUILD_COMMAND make
  INSTALL_COMMAND make install_sw
)

ExternalProject_Add(ext_libxml2
  URL "https://download.gnome.org/sources/libxml2/2.12/libxml2-2.12.7.tar.xz"
  URL_HASH SHA256=24ae78ff1363a973e6d8beba941a7945da2ac056e19b53956aeb6927fd6cfb56
  PREFIX build/deps/libxml2
  INSTALL_DIR ${CMAKE_INSTALL_PREFIX}
  CMAKE_ARGS -DCMAKE_INSTALL_PREFIX:PATH=<INSTALL_DIR> -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE} -DLIBXML2_WITH_PYTHON=OFF -DLIBXML2_WITH_LZMA=OFF -DLIBXML2_WITH_PROGRAMS=OFF -DLIBXML2_WITH_TESTS=OFF
)

ExternalProject_Add(ext_sqlite3
  URL "https://sqlite.org/2024/sqlite-autoconf-3460000.tar.gz"
  URL_HASH SHA256=6f8e6a7b335273748816f9b3b62bbdc372a889de8782d7f048c653a447417a7d
  PREFIX build/deps/sqlite3
  INSTALL_DIR ${CMAKE_INSTALL_PREFIX}
  BUILD_IN_SOURCE 1
  CONFIGURE_COMMAND ./configure --prefix=${CMAKE_INSTALL_PREFIX}
  BUILD_COMMAND make
  INSTALL_COMMAND make install
)

ExternalProject_Add(ext_sodium
  URL "https://github.com/jedisct1/libsodium/releases/download/1.0.20-RELEASE/libsodium-1.0.20.tar.gz"
  URL_HASH SHA256=ebb65ef6ca439333c2bb41a0c1990587288da07f6c7fd07cb3a18cc18d30ce19
  PREFIX build/deps/libsodium
  INSTALL_DIR ${CMAKE_INSTALL_PREFIX}
  BUILD_IN_SOURCE 1
  CONFIGURE_COMMAND ./configure --prefix=${CMAKE_INSTALL_PREFIX}
  BUILD_COMMAND make
  INSTALL_COMMAND make install
)

ExternalProject_Add(php81
  URL "https://www.php.net/distributions/php-8.1.29.tar.gz"
  URL_HASH SHA256=8b2609bf1d3173aa38269a9af21532c65f730aadd3051f9aae011eea9e246de5
  PREFIX build/deps/php81
  INSTALL_DIR ${CMAKE_INSTALL_PREFIX}
  BUILD_IN_SOURCE 1
  CONFIGURE_COMMAND ./configure "CFLAGS=-I${CMAKE_INSTALL_PREFIX}/include -I${MACOS_SDK_PREFIX}/usr/include ${CMAKE_C_FLAGS}" "LDFLAGS=-L${CMAKE_INSTALL_PREFIX}/lib -liconv ${CMAKE_EXE_LINKER_FLAGS}" "SQLITE_CFLAGS=''" "SQLITE_LIBS=-lsqlite3" "LIBXML_CFLAGS=-I${CMAKE_INSTALL_PREFIX}/libxml2" "LIBXML_LIBS=-liconv -lxml2" "OPENSSL_CFLAGS=''" "OPENSSL_LIBS=-L${CMAKE_INSTALL_PREFIX}/lib -lcrypto -lssl" "ZLIB_CFLAGS=''" "ZLIB_LIBS=-L${MACOS_SDK_PREFIX}/usr/lib -lz" "CURL_CFLAGS=''" "CURL_LIBS=-L${MACOS_SDK_PREFIX}/usr/lib -lcurl" "LIBSODIUM_CFLAGS=''" "LIBSODIUM_LIBS=-L${CMAKE_INSTALL_PREFIX}/lib -lsodium -lpthread" --prefix=${CMAKE_INSTALL_PREFIX} --with-openssl --enable-bcmath --with-zlib --with-curl --with-sodium
  BUILD_COMMAND make
  INSTALL_COMMAND make install
  DEPENDS ext_openssl ext_sodium ext_libxml2 ext_sqlite3
)
