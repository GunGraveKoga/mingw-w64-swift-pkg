
_compiler=clang
_compilerxx=clang++

_swift_clang_version=swift-4.1.3+mingw.20181009
_swift_llvm_version=swift-4.1.3+mingw.20181009
_swift_windows_version=swift-4.1.3+mingw.20181009
_swift_foundation_version=swift-4.1.3+mingw.20181009
_swift_xctest_version=swift-4.1.3+mingw.20181009

_build_dir=build/NinjaMinGW
_build_dir_cmark=("${_build_dir}/cmark")
_build_dir_llvm=("${_build_dir}/llvm")
_build_dir_swift=("${_build_dir}/swift")
_build_dir_xctest=("${_build_dir}/xctest-mingw-x86_64")


_realname=swift

pkgbase=mingw-w64-${_realname}
pkgname=("${MINGW_PACKAGE_PREFIX}-${_realname}")
pkgver=4.1.3.20181009
pkgrel=0
pkgdesc="The Swift Programming Language"
arch=('x86_64')
license=('Apache 2.0')
url="https://swift.org"

source=(
	"swift::git+https://github.com/tinysun212/swift-windows.git#tag=${_swift_windows_version}"
	"llvm::git+https://github.com/tinysun212/swift-llvm-cygwin.git#tag=${_swift_llvm_version}"
	"clang::git+https://github.com/tinysun212/swift-clang-cygwin.git#tag=${_swift_clang_version}"
	"cmark::git+https://github.com/apple/swift-cmark.git#branch=master"
	"swift-corelibs-foundation::git+https://github.com/tinysun212/swift-corelibs-foundation.git#tag=${_swift_foundation_version}"
	"swift-corelibs-xctest::git+https://github.com/tinysun212/swift-corelibs-xctest.git#tag=${_swift_xctest_version}"
	"swift-invoker.c"
)

noextract=()
md5sums=(
	'SKIP'
	'SKIP'
	'SKIP'
	'SKIP'
	'SKIP'
	'SKIP'
	'SKIP'
)

makedepends=(
	"git"
	"make"
	"${MINGW_PACKAGE_PREFIX}-ninja"
	"python"
	"python2"
	"${MINGW_PACKAGE_PREFIX}-pkg-config"
)

depends=(
	"${MINGW_PACKAGE_PREFIX}-binutils"
	"${MINGW_PACKAGE_PREFIX}-gcc"
	"${MINGW_PACKAGE_PREFIX}-clang"
	"${MINGW_PACKAGE_PREFIX}-icu"
	"${MINGW_PACKAGE_PREFIX}-libxml2"
	"${MINGW_PACKAGE_PREFIX}-wineditline"
	"${MINGW_PACKAGE_PREFIX}-winpthreads"
	"${MINGW_PACKAGE_PREFIX}-dlfcn"
	"${MINGW_PACKAGE_PREFIX}-cmake"
)

prepare() {

  	if [[ ! -d ${srcdir}/llvm/tools/clang ]]; then
  		cd ${srcdir}/llvm/tools
  		cmd /C "mklink /d clang ..\..\clang"
  		cd ${srcdir}
  	fi
}

build_cmark() {
	echo "Building cmark..."
	mkdir -p ${srcdir}/${_build_dir_cmark}
	cd ${srcdir}/${_build_dir_cmark}
	cmake -G Ninja -DCMAKE_BUILD_TYPE=RELEASE \
	-DCMAKE_C_COMPILER=${_compiler}  \
	-DCMAKE_CXX_COMPILER=${_compilerxx} ${srcdir}/cmark

	ninja
}

build_clang() {
	echo "Building swift-clang..."
	mkdir -p ${srcdir}/${_build_dir_llvm}
	cd ${srcdir}/${_build_dir_llvm}
	cmake -G Ninja -DCMAKE_BUILD_TYPE=RELEASE \
	-DCMAKE_C_COMPILER=${_compiler}  \
	-DCMAKE_CXX_COMPILER=${_compilerxx} \
	-DCMAKE_CXX_FLAGS="-femulated-tls" \
	-DLLVM_ENABLE_ASSERTIONS:BOOL=TRUE ${srcdir}/llvm

	ninja
}

build_swift() {
	echo "Building swift..."
	mkdir -p ${srcdir}/${_build_dir_swift}
	mkdir -p ${srcdir}/${_build_dir_swift}/bin
	cp -p ${srcdir}/${_build_dir_cmark}/src/libcmark.dll.a ${srcdir}/${_build_dir_swift}/libcmark.a
	cp -p ${srcdir}/${_build_dir_cmark}/src/libcmark.dll ${srcdir}/${_build_dir_swift}/bin/libcmark.dll
	cd ${srcdir}/${_build_dir_swift}
	cmake -G Ninja ${srcdir}/swift -DCMAKE_BUILD_TYPE=Release \
	-DPython_ADDITIONAL_VERSIONS=2.7 \
	-DCMAKE_C_COMPILER=${srcdir}/${_build_dir_llvm}/bin/clang.exe \
	-DCMAKE_CXX_COMPILER=${srcdir}/${_build_dir_llvm}/bin/clang++.exe \
	-DCMAKE_ASM_COMPILER=${srcdir}/${_build_dir_llvm}/bin/clang.exe \
	-DCMAKE_AR=${srcdir}/${_build_dir_llvm}/bin/llvm-ar.exe \
	-DCMAKE_RANLIB=${srcdir}/${_build_dir_llvm}/bin/llvm-ranlib.exe \
	-DPKG_CONFIG_EXECUTABLE=/mingw64/bin/pkg-config \
	-DICU_UC_INCLUDE_DIR=/mingw64/include \
	-DICU_UC_LIBRARY=/mingw64/lib/libicuuc.dll.a \
	-DICU_I18N_INCLUDE_DIR=/mingw64/include \
	-DICU_I18N_LIBRARY=/mingw64/lib/libicuin.dll.a \
	-DSWIFT_INCLUDE_DOCS=FALSE \
	-DSWIFT_PATH_TO_CMARK_BUILD=${srcdir}/${_build_dir_cmark} \
	-DSWIFT_PATH_TO_CMARK_SOURCE=${srcdir}/cmark \
	-DSWIFT_PATH_TO_LLVM_SOURCE=${srcdir}/llvm \
	-DSWIFT_PATH_TO_LLVM_BUILD=${srcdir}/${_build_dir_llvm} \
	-DSWIFT_STDLIB_ASSERTIONS:BOOL=TRUE \
	-DCMAKE_EXE_LINKER_FLAGS="-Xlinker --allow-multiple-definition" \
	-DCMAKE_SHARED_LINKER_FLAGS="-Xlinker --allow-multiple-definition" \
	-DCMAKE_MODULE_LINKER_FLAGS="-Xlinker --allow-multiple-definition" \
	${srcdir}/swift

	if [[ -f ${srcdir}/${_build_dir_swift}/lib/swift/mingw/x86_64/mingw_crt.modulemap ]]; then
		rm ${srcdir}/${_build_dir_swift}/lib/swift/mingw/x86_64/mingw_crt.modulemap
	fi

	ninja -j 4

	mv -f ${srcdir}/${_build_dir_swift}/lib/swift/mingw/x86_64/*.dll.a ${srcdir}/${_build_dir_swift}/lib/swift/mingw
}

build_foundation() {
	echo "Building Foundation..."
	export SWIFTC=${srcdir}/${_build_dir_swift}/bin/swiftc
	export SWIFT=${srcdir}/${_build_dir_swift}/bin/swift
	export CLANG=${srcdir}/${_build_dir_llvm}/bin/clang
	export CLANGXX=${srcdir}/${_build_dir_llvm}/bin/clang++
	export CFLAGS="-DWINDOWS_BOOL=WINBOOL"
	export BUILD_DIR=Build
	export DSTROOT=/
	export PREFIX=/usr
	export MSYSTEM=MINGW64
	export SDKROOT=${srcdir}/${_build_dir_swift}
	cd ${srcdir}/swift-corelibs-foundation
	/bin/python3 ./configure Release --target=x86_64-windows-gnu -DXCTEST_BUILD_DIR=${srcdir}/${_build_dir_xctest}
	export SDKROOT=

	if [[ ! -d Build/Foundation/CoreFoundation ]]; then
		mkdir -p Build/Foundation/CoreFoundation
		if [[ ! -d BFC ]]; then
			cmd /C "mklink /d BFC Build\Foundation\CoreFoundation"
		fi
	fi
	if [[ ! -d Build/Foundation/Foundation ]]; then
		mkdir -p Build/Foundation/Foundation
		if [[ ! -d BFF ]]; then
			cmd /C "mklink /d BFF Build\Foundation\Foundation"
		fi
	fi
	sed -i -e "s;Build/Foundation/CoreFoundation/;BFC/;g" \
		-e "s;Build/Foundation/Foundation/;BFF/;g" \
		build.ninja

	if [[ ! -z "$(ls -A Build/Foundation/Foundation)" ]]; then
		rm Build/Foundation/Foundation/*
	fi

	ninja

	mv -f ${srcdir}/swift-corelibs-foundation/Build/Foundation/libFoundation.a ${srcdir}/swift-corelibs-foundation/Build/Foundation/libFoundation_static.a
}

build_xctest() {
	echo "Building XCTest..."

	mkdir -p ${srcdir}/${_build_dir_xctest}
	cd  ${srcdir}/${_build_dir_xctest}

	if [[ ! -z "$(ls -A ${srcdir}/${_build_dir_xctest})" ]]; then
		rm ${srcdir}/${_build_dir_xctest}/*
	fi

	${srcdir}/${_build_dir_swift}/bin/swiftc -Xcc -fblocks -c -O -module-name XCTest -module-link-name XCTest -parse-as-library \
		-I/mingw64/include \
		-I/mingw64/x86_64-w64-mingw32/include \
		-I${srcdir}/swift-corelibs-foundation/Build/Foundation/usr/lib/swift \
		-I${srcdir}/swift-corelibs-foundation/Build/Foundation \
		-emit-module-path ${srcdir}/${_build_dir_xctest}/XCTest.swiftmodule \
		-force-single-frontend-invocation \
		-swift-version 4 \
		${srcdir}/swift-corelibs-xctest/Sources/XCTest/Private/*.swift \
		${srcdir}/swift-corelibs-xctest/Sources/XCTest/Public//Asynchronous/*.swift \
		${srcdir}/swift-corelibs-xctest/Sources/XCTest/Public/*.swift \
		-o ${srcdir}/${_build_dir_xctest}/XCTest.o

	${srcdir}/${_build_dir_swift}/bin/swiftc -emit-library ${srcdir}/${_build_dir_xctest}/XCTest.o \
		-lswiftCore -L${srcdir}/swift-corelibs-foundation/Build/Foundation -lFoundation -lm \
		-Xlinker --allow-multiple-definition -licuin -lxml2 -licuuc \
		-o ${srcdir}/${_build_dir_xctest}/libXCTest.dll
}

build_swift-invoker() {
	echo "Building swift-invoker..."
	${_compiler} -o ${srcdir}/${_build_dir}/swift-invoker.exe -Wall ${srcdir}/swift-invoker.c
	strip ${srcdir}/${_build_dir}/swift-invoker.exe
}

build() {
	build_cmark
	build_clang
	build_swift
	build_foundation
	build_xctest
	build_swift-invoker
}

package_mingw-w64-x86_64-swift() {
	echo "Packing swift..."
	mkdir -p ${pkgdir}${MINGW_PREFIX}/bin
	mkdir -p ${pkgdir}${MINGW_PREFIX}/lib
	mkdir -p ${pkgdir}${MINGW_PREFIX}/lib/pkgconfig
	mkdir -p ${pkgdir}${MINGW_PREFIX}/lib/swift

	sed -i -e 's;c:/msys64;../../../../..;' ${srcdir}/${_build_dir_swift}/lib/swift/mingw/x86_64/mingw_crt.modulemap

	cp -p ${srcdir}/${_build_dir_cmark}/src/cmark.exe ${pkgdir}${MINGW_PREFIX}/bin
	cp -p ${srcdir}/${_build_dir_cmark}/src/libcmark.dll ${pkgdir}${MINGW_PREFIX}/bin
	cp -p ${srcdir}/${_build_dir_cmark}/src/libcmark.dll.a ${pkgdir}${MINGW_PREFIX}/lib
	cp -p ${srcdir}/${_build_dir_cmark}/src/libcmark.a ${pkgdir}${MINGW_PREFIX}/lib
	cp -p ${srcdir}/${_build_dir_cmark}/src/libcmark.pc ${pkgdir}${MINGW_PREFIX}/lib/pkgconfig
	cp -p ${srcdir}/${_build_dir_swift}/bin/swift.exe ${pkgdir}${MINGW_PREFIX}/bin
	cp -p ${srcdir}/${_build_dir_swift}/bin/swift-demangle.exe ${pkgdir}${MINGW_PREFIX}/bin
	cp -p ${srcdir}/${_build_dir_swift}/bin/swift-ide-test.exe ${pkgdir}${MINGW_PREFIX}/bin
	cp -p ${srcdir}/${_build_dir}/swift-invoker.exe ${pkgdir}${MINGW_PREFIX}/bin/swiftc.exe
	cp -p ${srcdir}/${_build_dir}/swift-invoker.exe ${pkgdir}${MINGW_PREFIX}/bin/swift-autolink-extract.exe
	cp -p ${srcdir}/${_build_dir_swift}/bin/*.dll ${pkgdir}${MINGW_PREFIX}/bin
	cp -rp ${srcdir}/${_build_dir_swift}/lib/swift/clang ${pkgdir}${MINGW_PREFIX}/lib/swift
	cp -rp ${srcdir}/${_build_dir_swift}/lib/swift/mingw ${pkgdir}${MINGW_PREFIX}/lib/swift
	cp -rp ${srcdir}/${_build_dir_swift}/lib/swift/shims ${pkgdir}${MINGW_PREFIX}/lib/swift

	echo "Packing swift-corelibs-foundation..."

	cp -p ${srcdir}/swift-corelibs-foundation/Build/Foundation/libFoundation.dll ${pkgdir}${MINGW_PREFIX}/bin
	cp -p ${srcdir}/swift-corelibs-foundation/Build/Foundation/libFoundation.dll ${pkgdir}${MINGW_PREFIX}/lib/swift/mingw
	cp -rp ${srcdir}/swift-corelibs-foundation/Build/Foundation/usr/lib/swift/CoreFoundation ${pkgdir}${MINGW_PREFIX}/lib/swift
	cp -p ${srcdir}/swift-corelibs-foundation/Build/Foundation/Foundation.swiftdoc ${pkgdir}${MINGW_PREFIX}/lib/swift/mingw/x86_64
	cp -p ${srcdir}/swift-corelibs-foundation/Build/Foundation/Foundation.swiftmodule ${pkgdir}${MINGW_PREFIX}/lib/swift/mingw/x86_64
	cp -rp ${srcdir}/swift-corelibs-foundation/CoreFoundation/CharacterSets ${pkgdir}${MINGW_PREFIX}/bin

	echo "Packing swift-corelibs-xctest..."

	cp -p ${srcdir}/${_build_dir_xctest}/libXCTest.dll ${pkgdir}${MINGW_PREFIX}/bin
	cp -p ${srcdir}/${_build_dir_xctest}/libXCTest.dll ${pkgdir}${MINGW_PREFIX}/lib/swift/mingw
	cp -p ${srcdir}/${_build_dir_xctest}/XCTest.swiftmodule ${pkgdir}${MINGW_PREFIX}/lib/swift/mingw/x86_64
	cp -p ${srcdir}/${_build_dir_xctest}/XCTest.swiftdoc ${pkgdir}${MINGW_PREFIX}/lib/swift/mingw/x86_64
}