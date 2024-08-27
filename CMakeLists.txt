cmake_minimum_required(VERSION 3.7)

if (CMAKE_VERSION VERSION_GREATER_EQUAL "3.24.0")
	cmake_policy(SET CMP0135 NEW)
endif()

project(macphp LANGUAGES C)

include(ExternalProject)

execute_process(COMMAND xcrun --show-sdk-path OUTPUT_VARIABLE MACOS_SDK_PREFIX OUTPUT_STRIP_TRAILING_WHITESPACE)

ExternalProject_Add(ext_ssl
  URL "https://github.com/libressl/portable/releases/download/v3.9.2/libressl-3.9.2.tar.gz"
  URL_HASH SHA256=7b031dac64a59eb6ee3304f7ffb75dad33ab8c9d279c847f92c89fb846068f97
  PREFIX build/deps/ssl
  INSTALL_DIR ${CMAKE_INSTALL_PREFIX}
  CMAKE_ARGS -DCMAKE_INSTALL_PREFIX:PATH=<INSTALL_DIR> -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE} -DLIBRESSL_APPS=OFF -DLIBRESSL_TESTS=OFF -DENABLE_ASM=OFF
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

ExternalProject_Add(ext_onigiruma
  URL "https://github.com/kkos/oniguruma/releases/download/v6.9.9/onig-6.9.9.tar.gz"
  URL_HASH SHA256=60162bd3b9fc6f4886d4c7a07925ffd374167732f55dce8c491bfd9cd818a6cf
  PREFIX build/deps/oniguruma
  INSTALL_DIR ${CMAKE_INSTALL_PREFIX}
  CMAKE_ARGS -DCMAKE_INSTALL_PREFIX:PATH=<INSTALL_DIR> -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}
)

ExternalProject_Add(php81
  URL "https://www.php.net/distributions/php-8.1.29.tar.gz"
  URL_HASH SHA256=8b2609bf1d3173aa38269a9af21532c65f730aadd3051f9aae011eea9e246de5
  PREFIX build/deps/php81
  INSTALL_DIR ${CMAKE_INSTALL_PREFIX}
  BUILD_IN_SOURCE 1
  CONFIGURE_COMMAND ./configure "CFLAGS=-I${CMAKE_INSTALL_PREFIX}/include -I${MACOS_SDK_PREFIX}/usr/include ${CMAKE_C_FLAGS}" "LDFLAGS=-L${CMAKE_INSTALL_PREFIX}/lib -liconv ${CMAKE_EXE_LINKER_FLAGS}" "SQLITE_CFLAGS=''" "SQLITE_LIBS=-lsqlite3" "LIBXML_CFLAGS=''" "LIBXML_LIBS=-liconv -lxml2" "OPENSSL_CFLAGS=''" "OPENSSL_LIBS=-L${CMAKE_INSTALL_PREFIX}/lib -ltls -lcrypto -lssl" "ZLIB_CFLAGS=''" "ZLIB_LIBS=-L${MACOS_SDK_PREFIX}/usr/lib -lz" "CURL_CFLAGS=''" "CURL_LIBS=-L${MACOS_SDK_PREFIX}/usr/lib -lcurl" "LIBSODIUM_CFLAGS=''" "LIBSODIUM_LIBS=-L${CMAKE_INSTALL_PREFIX}/lib -lsodium -lpthread" "ONIG_CFLAGS=''" "ONIG_LIBS=-L${CMAKE_INSTALL_PREFIX}/lib -lonig" --prefix=${CMAKE_INSTALL_PREFIX} --with-openssl --enable-bcmath=shared --enable-mbstring=shared --with-zlib --with-curl --with-sodium
  BUILD_COMMAND make
  INSTALL_COMMAND make install
  DEPENDS ext_ssl ext_sodium ext_onigiruma
)

ExternalProject_Add(m4
  URL "https://ftp.gnu.org/gnu/m4/m4-1.4.19.tar.gz"
  URL_HASH SHA256=3be4a26d825ffdfda52a56fc43246456989a3630093cced3fbddf4771ee58a70
  PREFIX build/deps/m4
  INSTALL_DIR ${CMAKE_INSTALL_PREFIX}
  BUILD_IN_SOURCE 1
  CONFIGURE_COMMAND ./configure --prefix=${CMAKE_INSTALL_PREFIX}
  BUILD_COMMAND make
  INSTALL_COMMAND make install
)

ExternalProject_Add(autoconf
  URL "https://ftp.gnu.org/gnu/autoconf/autoconf-2.72.tar.gz"
  URL_HASH SHA256=afb181a76e1ee72832f6581c0eddf8df032b83e2e0239ef79ebedc4467d92d6e
  PREFIX build/deps/autoconf
  INSTALL_DIR ${CMAKE_INSTALL_PREFIX}
  BUILD_IN_SOURCE 1
  CONFIGURE_COMMAND ./configure --prefix=${CMAKE_INSTALL_PREFIX} M4=${CMAKE_INSTALL_PREFIX}/bin/m4
  BUILD_COMMAND make
  INSTALL_COMMAND make install
  DEPENDS m4
)

ExternalProject_Add(xdebug
  URL "https://xdebug.org/files/xdebug-3.3.2.tgz"
  URL_HASH SHA256=15545b8311bedb5e7df08fd90d6653d9bf33629791e8119e70350475df932c55
  PREFIX build/deps/xdebug/xdebug-3.3.2
  INSTALL_DIR ${CMAKE_INSTALL_PREFIX}
  BUILD_IN_SOURCE 1
  CONFIGURE_COMMAND cd xdebug-3.3.2 && PHP_AUTOCONF=${CMAKE_INSTALL_PREFIX}/bin/autoconf PHP_AUTOHEADER=${CMAKE_INSTALL_PREFIX}/bin/autoheader phpize && ./configure --prefix=${CMAKE_INSTALL_PREFIX}
  BUILD_COMMAND cd xdebug-3.3.2 && make
  INSTALL_COMMAND cd xdebug-3.3.2 && make install
  DEPENDS php81 autoconf
)
