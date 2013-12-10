# XCODE_DEVELOPER = $(shell xcode-select --print-path)
# IOS_PLATFORM ?= iPhoneOS
NDK_BASE ?= /android-ndk-r8d
STANDALONE_BASE_ARM = ${CURDIR}/standalone-toolchain-arm
STANDALONE_BASE_X86 = ${CURDIR}/standalone-toolchain-x86
HOST_ARM = arm-linux-androideabi
HOST_X86 = i686-linux-android
TOOLCHAIN_ARM = arm-linux-androideabi-4.6
TOOLCHAIN_X86 = x86-4.6
ifeq (${ARCH},armeabi)
	STANDALONE_BASE = ${STANDALONE_BASE_ARM}
	HOST = ${HOST_ARM}
	TOOLCHAIN = ${TOOLCHAIN_ARM}
	CFLAGS_1 = -mthumb -march=armv5te
	LDFLAGS_1 = -Wl,--fix-cortex-a8
else ifeq (${ARCH},armeabi-v7a)
	STANDALONE_BASE = ${STANDALONE_BASE_ARM}
	HOST = ${HOST_ARM}
	TOOLCHAIN = ${TOOLCHAIN_ARM}
	CFLAGS_1 = -mthumb -march=armv7-a -mfloat-abi=softfp
	LDFLAGS_1 = -Wl,--fix-cortex-a8
else ifeq (${ARCH},x86)
	STANDALONE_BASE = ${STANDALONE_BASE_X86}
	HOST = ${HOST_X86}
	TOOLCHAIN = ${TOOLCHAIN_X86}
	CFLAGS_1 = 
	LDFLAGS_1 = 
endif
PLATFORM ?= android-14

# Pick latest SDK in the directory
#IOS_PLATFORM_DEVELOPER = ${XCODE_DEVELOPER}/Platforms/${IOS_PLATFORM}.platform/Developer
#IOS_SDK = ${IOS_PLATFORM_DEVELOPER}/SDKs/$(shell ls ${IOS_PLATFORM_DEVELOPER}/SDKs | sort -r | head -n1)

#all: lib/libspatialite.a
all: create_standalone

create_standalone:
	${NDK_BASE}/build/tools/make-standalone-toolchain.sh --toolchain=${TOOLCHAIN_ARM} --platform=${PLATFORM} --install-dir=${STANDALONE_BASE_ARM}
	${NDK_BASE}/build/tools/make-standalone-toolchain.sh --toolchain=${TOOLCHAIN_X86} --platform=${PLATFORM} --install-dir=${STANDALONE_BASE_X86}
	${MAKE} build_arches

#lib/libspatialite.a: build_arches
#	mkdir -p lib
#	mkdir -p include
#
#	# Copy includes
#	cp -R build/armv7/include/geos include
#	cp -R build/armv7/include/spatialite include
#	cp -R build/armv7/include/*.h include
#
#	# Make fat libraries for all architectures
#	for file in build/armv7/lib/*.a; \
#		do name=`basename $$file .a`; \
#		${IOS_PLATFORM_DEVELOPER}/usr/bin/lipo -create \
#			-arch armv7 build/armv7/lib/$$name.a \
#			-arch armv7s build/armv7s/lib/$$name.a \
#			-arch i386 build/i386/lib/$$name.a \
#			-output lib/$$name.a \
#		; \
#		done;

# Build separate architectures
build_arches:
	${MAKE} arch ARCH=armeabi
	${MAKE} arch ARCH=armeabi-v7a
	${MAKE} arch ARCH=x86

PREFIX = ${CURDIR}/build/${ARCH}
LIBDIR = ${PREFIX}/lib
BINDIR = ${PREFIX}/bin
INCLUDEDIR = ${PREFIX}/include

STRIP = ${STANDALONE_BASE}/bin/${HOST}-strip
CC = ${STANDALONE_BASE}/bin/${HOST}-gcc
LD = ${STANDALONE_BASE}/bin/${HOST}-ld
AR = ${STANDALONE_BASE}/bin/${HOST}-ar
RANLIB = ${STANDALONE_BASE}/bin/${HOST}-ranlib
OBJDUMP = ${STANDALONE_BASE}/bin/${HOST}-objdump
CXX = ${STANDALONE_BASE}/bin/${HOST}-g++
CPP = ${STANDALONE_BASE}/bin/${HOST}-cpp

CFLAGS = ${CFLAGS_1} -I${INCLUDEDIR} -I${STANDALONE_BASE}/include -ffunction-sections -funwind-tables -fstack-protector -no-canonical-prefixes -fomit-frame-pointer -fno-strict-aliasing -finline-limit=64 -DANDROID -D__ANDROID__ -Wa,--noexecstack
CXXFLAGS = -ffunction-sections -funwind-tables -no-canonical-prefixes -fstack-protector -fomit-frame-pointer -fstrict-aliasing -funswitch-loops -finline-limit=300 -DANDROID -D__ANDROID__ -Wa,--noexecstack -frtti -fexceptions
CPPFLAGS = -I${INCLUDEDIR} -I${STANDALONE_BASE}/include -DANDROID -D__ANDROID__
LDFLAGS = ${LDFLAGS_1} -L${LIBDIR} -L${STANDALONE_BASE}/lib

#CFLAGS = -isysroot ${IOS_SDK} -I${IOS_SDK}/usr/include -arch ${ARCH} -I${INCLUDEDIR} -miphoneos-version-min=5.0
#CXXFLAGS = -stdlib=libc++ -std=c++11 -isysroot ${IOS_SDK} -I${IOS_SDK}/usr/include -arch ${ARCH} -I${INCLUDEDIR} -miphoneos-version-min=5.0
#LDFLAGS = -stdlib=libc++ -isysroot ${IOS_SDK} -L${LIBDIR} -L${IOS_SDK}/usr/lib -arch ${ARCH} -miphoneos-version-min=5.0

arch: ${LIBDIR}/libspatialite.a
#arch: ${LIBDIR}/libiconv.a

${LIBDIR}/libspatialite.a: ${LIBDIR}/libproj.a ${LIBDIR}/libgeos.a ${LIBDIR}/libsqlite3.a ${LIBDIR}/libiconv.a ${CURDIR}/spatialite
	cd spatialite && env \
	STRIP=${STRIP} \
	CXX=${CXX} \
	CC=${CC} \
	CPP=${CPP} \
	LD=${LD} \
	AR=${AR} \
	RANLIB=${RANLIB} \
	OBJDUMP=${OBJDUMP} \
	CFLAGS="${CFLAGS}" \
	CXXFLAGS="${CXXFLAGS}" \
	CPPFLAGS="${CPPFLAGS}" \
	LDFLAGS="${LDFLAGS} -liconv -lgeos -lgeos_c" ./configure --host=${HOST} --disable-freexl --prefix=${PREFIX} --with-geosconfig=${BINDIR}/geos-config --disable-shared && make clean install-strip

${CURDIR}/spatialite:
	curl http://www.gaia-gis.it/gaia-sins/libspatialite-4.1.1.tar.gz > spatialite.tar.gz
	tar -xzf spatialite.tar.gz
	rm spatialite.tar.gz
	mv libspatialite-4.1.1 spatialite
	cp -rf patches/config.* spatialite/.
	patch -u ${CURDIR}/spatialite/src/Makefile.in < ${CURDIR}/patches/spatialite.diff

${LIBDIR}/libiconv.a: ${CURDIR}/iconv
	cd iconv && env \
	STRIP=${STRIP} \
	CXX=${CXX} \
	CC=${CC} \
	CPP=${CPP} \
	LD=${LD} \
	AR=${AR} \
	RANLIB=${RANLIB} \
	OBJDUMP=${OBJDUMP} \
	CFLAGS="${CFLAGS} -O2" \
	CXXFLAGS="${CXXFLAGS}" \
	CPPFLAGS="${CPPFLAGS}" \
	LDFLAGS="${LDFLAGS} -s" gl_cv_header_working_stdint_h=yes ./configure --host=${HOST} --prefix=${PREFIX} --disable-shared && make clean install

${CURDIR}/iconv:
	curl http://ftp.gnu.org/gnu/libiconv/libiconv-1.14.tar.gz > iconv.tar.gz
	tar -xzf iconv.tar.gz
	rm iconv.tar.gz
	mv libiconv-1.14 iconv
	cp -rf patches/config.* iconv/build-aux/.
	cp -rf patches/config.* iconv/libcharset/build-aux/.
	patch -u ${CURDIR}/iconv/Makefile.in < ${CURDIR}/patches/iconv.diff 

${LIBDIR}/libproj.a: ${CURDIR}/proj
	cd proj && env \
	STRIP=${STRIP} \
	CXX=${CXX} \
	CC=${CC} \
	CPP=${CPP} \
	LD=${LD} \
	AR=${AR} \
	RANLIB=${RANLIB} \
	OBJDUMP=${OBJDUMP} \
	CFLAGS="${CFLAGS}" \
	CXXFLAGS="${CXXFLAGS}" \
	CPPFLAGS="${CPPFLAGS}" \
	LDFLAGS="${LDFLAGS}" ./configure --host=${HOST} --with-jni=no --prefix=${PREFIX} --disable-shared && make clean install

${CURDIR}/proj:
	curl http://download.osgeo.org/proj/proj-4.8.0.tar.gz > proj.tar.gz
	tar -xzf proj.tar.gz
	rm proj.tar.gz
	mv proj-4.8.0 proj
	cp -rf patches/config.* proj/.
	patch -p3 ${CURDIR}/proj/src/pj_init.c < ${CURDIR}/patches/proj.diff

${LIBDIR}/libgeos.a: ${CURDIR}/geos
	cd geos && env \
	STRIP=${STRIP} \
	CXX=${CXX} \
	CC=${CC} \
	CPP=${CPP} \
	LD=${LD} \
	AR=${AR} \
	RANLIB=${RANLIB} \
	OBJDUMP=${OBJDUMP} \
	CFLAGS="${CFLAGS}" \
	CXXFLAGS="${CXXFLAGS}" \
	CPPFLAGS="${CPPFLAGS}" \
	LDFLAGS="${LDFLAGS}" ./configure --host=${HOST} --prefix=${PREFIX} --disable-shared && \
	patch -u ${CURDIR}/geos/include/geos/platform.h < ${CURDIR}/patches/geos.diff && make clean install

${CURDIR}/geos:
	curl http://download.osgeo.org/geos/geos-3.4.2.tar.bz2 > geos.tar.bz2
	tar -xzf geos.tar.bz2
	rm geos.tar.bz2
	mv geos-3.4.2 geos
	cp -rf patches/config.* geos/.

${LIBDIR}/libsqlite3.a: ${CURDIR}/sqlite3
	cd sqlite3 && env \
	CXX=${CXX} \
	CC=${CC} \
	CPP=${CPP} \
	LD=${LD} \
	AR=${AR} \
	RANLIB=${RANLIB} \
	OBJDUMP=${OBJDUMP} \
	CFLAGS="${CFLAGS}" \
	CXXFLAGS="${CXXFLAGS}" \
	CPPFLAGS="${CPPFLAGS}" \
	CFLAGS="${CFLAGS} -DSQLITE_THREADSAFE=1 -DSQLITE_ENABLE_RTREE=1 -DSQLITE_ENABLE_FTS3=1 -DSQLITE_ENABLE_FTS3_PARENTHESIS=1" \
	CXXFLAGS="${CXXFLAGS} -DSQLITE_THREADSAFE=1 -DSQLITE_ENABLE_RTREE=1 -DSQLITE_ENABLE_FTS3=1 -DSQLITE_ENABLE_FTS3_PARENTHESIS=1" \
	LDFLAGS="${LDFLAGS}" ./configure --host=${HOST} --prefix=${PREFIX} --disable-shared --enable-static && make clean install

${CURDIR}/sqlite3:
	curl http://sqlite.org/2013/sqlite-autoconf-3080100.tar.gz > sqlite3.tar.gz
	tar xzvf sqlite3.tar.gz
	rm sqlite3.tar.gz
	mv sqlite-autoconf-3080100 sqlite3
	cp -rf patches/config.* sqlite3/.
	touch sqlite3

clean:
	rm -rf build geos proj spatialite sqlite3 iconv include lib ${STANDALONE_BASE_ARM} ${STANDALONE_BASE_X86}
