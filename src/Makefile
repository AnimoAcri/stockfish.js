# Stockfish, a UCI chess playing engine derived from Glaurung 2.1
# Copyright (C) 2004-2008 Tord Romstad (Glaurung author)
# Copyright (C) 2008-2015 Marco Costalba, Joona Kiiski, Tord Romstad
# Copyright (C) 2015-2019 Marco Costalba, Joona Kiiski, Gary Linscott, Tord Romstad
#
# Stockfish is free software: you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation, either version 3 of the License, or
# (at your option) any later version.
#
# Stockfish is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program.  If not, see <http://www.gnu.org/licenses/>.


### ==========================================================================
### Section 1. General Configuration
### ==========================================================================

### Executable name
ifeq ($(COMP),mingw)
EXE = stockfish.exe
else
EXE = stockfish
endif

### Installation dir definitions
PREFIX = /usr/local
BINDIR = $(PREFIX)/bin

### Built-in benchmark for pgo-builds
PGOBENCH = ./$(EXE) bench

### Object files
OBJS = bitbase.o bitboard.o endgame.o evaluate.o main.o \
	material.o misc.o movegen.o movepick.o pawns.o position.o psqt.o \
	search.o thread.o timeman.o tt.o uci.o ucioption.o
ifneq ($(COMP),emscripten)
	OBJS += benchmark.o
	OBJS += syzygy/tbprobe.o
endif

### Establish the operating system name
KERNEL = $(shell uname -s)
ifeq ($(KERNEL),Linux)
	OS = $(shell uname -o)
endif

### ==========================================================================
### Section 2. High-level Configuration
### ==========================================================================
#
# flag                --- Comp switch --- Description
# ----------------------------------------------------------------------------
#
# debug = yes/no      --- -DNDEBUG         --- Enable/Disable debug mode
# sanitize = undefined/thread/no (-fsanitize )
#                     --- ( undefined )    --- enable undefined behavior checks
#                     --- ( thread    )    --- enable threading error  checks
# optimize = yes/no   --- (-O3/-fast etc.) --- Enable/Disable optimizations
# arch = (name)       --- (-arch)          --- Target architecture
# bits = 64/32        --- -DIS_64BIT       --- 64-/32-bit operating system
# prefetch = yes/no   --- -DUSE_PREFETCH   --- Use prefetch asm-instruction
# popcnt = yes/no     --- -DUSE_POPCNT     --- Use popcnt asm-instruction
# sse = yes/no        --- -msse            --- Use Intel Streaming SIMD Extensions
# pext = yes/no       --- -DUSE_PEXT       --- Use pext x86_64 asm-instruction
#
# Note that Makefile is space sensitive, so when adding new architectures
# or modifying existing flags, you have to make sure there are no extra spaces
# at the end of the line for flag values.

### 2.1. General and architecture defaults
optimize = yes
debug = no
sanitize = no
bits = 32
prefetch = no
popcnt = no
sse = no
pext = no

### 2.2 Architecture specific

ifeq ($(ARCH),general-32)
	arch = any
endif

ifeq ($(ARCH),x86-32-old)
	arch = i386
endif

ifeq ($(ARCH),x86-32)
	arch = i386
	prefetch = yes
	sse = yes
endif

ifeq ($(ARCH),general-64)
	arch = any
	bits = 64
endif

ifeq ($(ARCH),asmjs)
	arch = any
	os = any
	bits = 32
	prefetch = no
	bsfq = no
	popcnt = no
	sse = no
	COMP = emscripten
	EXE = stockfish.asm.js
endif

ifeq ($(ARCH),wasm)
	arch = any
	os = any
	bits = 32
	prefetch = no
	bsfq = no
	popcnt = no
	sse = no
	COMP = emscripten
	EXE = stockfish.js
endif

ifeq ($(ARCH),x86-64)
	arch = x86_64
	bits = 64
	prefetch = yes
	sse = yes
endif

ifeq ($(ARCH),x86-64-modern)
	arch = x86_64
	bits = 64
	prefetch = yes
	popcnt = yes
	sse = yes
endif

ifeq ($(ARCH),x86-64-bmi2)
	arch = x86_64
	bits = 64
	prefetch = yes
	popcnt = yes
	sse = yes
	pext = yes
endif

ifeq ($(ARCH),armv7)
	arch = armv7
	prefetch = yes
endif

ifeq ($(ARCH),ppc-32)
	arch = ppc
endif

ifeq ($(ARCH),ppc-64)
	arch = ppc64
	bits = 64
endif


### ==========================================================================
### Section 3. Low-level configuration
### ==========================================================================

### 3.1 Selecting compiler (default = gcc)

ifeq ($(STATIC),1)
	CXXFLAGS += -static
endif

CXXFLAGS += -Wall -Wcast-qual -fno-exceptions -std=c++11 $(EXTRACXXFLAGS)
DEPENDFLAGS += -std=c++11
LDFLAGS += $(EXTRALDFLAGS)

ifeq ($(COMP),)
	COMP=gcc
endif

ifeq ($(COMP),gcc)
	comp=gcc
	CXX=g++
	CXXFLAGS += -pedantic -Wextra -Wshadow

	ifeq ($(ARCH),armv7)
		ifeq ($(OS),Android)
			CXXFLAGS += -m$(bits)
			LDFLAGS += -m$(bits)
		endif
	else
		CXXFLAGS += -m$(bits)
		LDFLAGS += -m$(bits)
	endif

	ifneq ($(KERNEL),Darwin)
	   LDFLAGS += -Wl,--no-as-needed
	endif
endif

ifeq ($(COMP),mingw)
	comp=mingw

	ifeq ($(KERNEL),Linux)
		ifeq ($(bits),64)
			ifeq ($(shell which x86_64-w64-mingw32-c++-posix),)
				CXX=x86_64-w64-mingw32-c++
			else
				CXX=x86_64-w64-mingw32-c++-posix
			endif
		else
			ifeq ($(shell which i686-w64-mingw32-c++-posix),)
				CXX=i686-w64-mingw32-c++
			else
				CXX=i686-w64-mingw32-c++-posix
			endif
		endif
	else
		CXX=g++
	endif

	CXXFLAGS += -Wextra -Wshadow
	LDFLAGS += -static
endif

ifeq ($(COMP),icc)
	comp=icc
	CXX=icpc
	CXXFLAGS += -diag-disable 1476,10120 -Wcheck -Wabi -Wdeprecated -strict-ansi
endif

ifeq ($(COMP),emscripten)
	comp=clang
	CXX=em++
endif

ifeq ($(COMP),clang)
	comp=clang
	CXX=clang++
	CXXFLAGS += -pedantic -Wextra -Wshadow

	ifneq ($(KERNEL),Darwin)
	ifneq ($(KERNEL),OpenBSD)
		LDFLAGS += -latomic
	endif
	endif

	ifeq ($(ARCH),armv7)
		ifeq ($(OS),Android)
			CXXFLAGS += -m$(bits)
			LDFLAGS += -m$(bits)
		endif
	else
		CXXFLAGS += -m$(bits)
		LDFLAGS += -m$(bits)
	endif
endif

ifeq ($(comp),icc)
	profile_make = icc-profile-make
	profile_use = icc-profile-use
else
ifeq ($(comp),clang)
	profile_make = clang-profile-make
	profile_use = clang-profile-use
else
	profile_make = gcc-profile-make
	profile_use = gcc-profile-use
endif
endif

ifneq ($(COMP),emscripten)
ifeq ($(KERNEL),Darwin)
	CXXFLAGS += -arch $(arch) -mmacosx-version-min=10.9
	LDFLAGS += -arch $(arch) -mmacosx-version-min=10.9
endif
endif

### Travis CI script uses COMPILER to overwrite CXX
ifdef COMPILER
	COMPCXX=$(COMPILER)
endif

ifeq ($(COMP),emscripten)
    ifeq ($(ARCH),wasm)
        CXXFLAGS += -s WASM=1 -s BINARYEN_ASYNC_COMPILATION=1
        LDFLAGS += -s WASM=1 -s BINARYEN_ASYNC_COMPILATION=1
        CXXFLAGS += -DWASM
    else
        CXXFLAGS += -s WASM=0
        LDFLAGS += -s WASM=0 -s LEGACY_VM_SUPPORT=1
        CXXFLAGS += -DASMJS
	endif
	CXXFLAGS += -s TOTAL_MEMORY=67108864 -s NO_FILESYSTEM=1 -s ERROR_ON_UNDEFINED_SYMBOLS=0 -s LINKABLE=1 -s EXPORT_ALL=1
	#NOTE: --closure 1 breaks the code
	#TODO: File bug report for --closure 1.
	LDFLAGS += -s TOTAL_MEMORY=67108864 -s NO_FILESYSTEM=1 -s EXPORTED_FUNCTIONS="['_init', '_uci_command', '_main']" -s "EXTRA_EXPORTED_RUNTIME_METHODS=['ccall']" --memory-init-file 0 -s NO_EXIT_RUNTIME=1 --pre-js pre.js --post-js post.js -s LINKABLE=1 -s EXPORT_ALL=1
endif
ifeq ($(CHESSCOM),1)
	CXXFLAGS += -DCHESSCOM
	#NOTE: uci.cpp uses shadowing for simplicity.
	CXXFLAGS:=$(filter-out -Wshadow,$(CXXFLAGS))
endif

### Allow overwriting CXX from command line
ifdef COMPCXX
	CXX=$(COMPCXX)
endif

### On mingw and emscripten do not use POSIX threads
ifneq ($(COMP),emscripten)
ifneq ($(comp),mingw)
	# On Android Bionic's C library comes with its own pthread implementation bundled in
	ifneq ($(OS),Android)
		# Haiku has pthreads in its libroot, so only link it in on other platforms
		ifneq ($(KERNEL),Haiku)
			LDFLAGS += -lpthread
		endif
	endif
endif
endif # end if for JS.

### 3.2.1 Debugging
ifneq ($(VARIANTS),NONE)
	ifeq ($(VARIANTS),)
		CXXFLAGS += -DANTI -DATOMIC -DCRAZYHOUSE -DHORDE -DKOTH -DRACE -DRELAY -DTHREECHECK
		#CXXFLAGS += -DANTI -DATOMIC -DBUGHOUSE -DCRAZYHOUSE -DDISPLACEDGRID -DEXTINCTION -DGRID -DHORDE -DKOTH -DLOOP -DLOSERS -DPLACEMENT -DRACE -DSLIPPEDGRID -DSUICIDE -DTHREECHECK -DTWOKINGS -DTWOKINGSSYMMETRIC
	else
		ifeq ($(findstring ANTI,$(VARIANTS)), ANTI)
			CXXFLAGS += -DANTI
		endif
		ifeq ($(findstring ATOMIC,$(VARIANTS)), ATOMIC)
			CXXFLAGS += -DATOMIC
		endif
		ifeq ($(findstring CRAZYHOUSE,$(VARIANTS)), CRAZYHOUSE)
			CXXFLAGS += -DCRAZYHOUSE
		endif
		ifeq ($(findstring HORDE,$(VARIANTS)), HORDE)
			CXXFLAGS += -DHORDE
		endif
		ifeq ($(findstring KOTH,$(VARIANTS)), KOTH)
			CXXFLAGS += -DKOTH
		endif
		# alternate name for king of the hill
		ifeq ($(findstring KINGOFTHEHILL,$(VARIANTS)), KINGOFTHEHILL)
			CXXFLAGS += -DKOTH
		endif
		ifeq ($(findstring RACE,$(VARIANTS)), RACE)
			CXXFLAGS += -DRACE
		endif
		ifeq ($(findstring RELAY,$(VARIANTS)), RELAY)
			CXXFLAGS += -DRELAY
		endif
		ifeq ($(findstring THREECHECK,$(VARIANTS)), THREECHECK)
			CXXFLAGS += -DTHREECHECK
		endif
		# alternate name for 3check
		ifeq ($(findstring 3CHECK,$(VARIANTS)), 3CHECK)
			CXXFLAGS += -DTHREECHECK
		endif
		
		ifneq (,$(filter -DBUGHOUSE,$(CXXFLAGS)))
			ifeq (,$(filter -DCRAZYHOUSE,$(CXXFLAGS)))
				$(error Crazyhouse (-DCRAZYHOUSE) is required for subvariant bughouse chess)
			endif
		endif
		ifneq (,$(filter -DDISPLACEDGRID,$(CXXFLAGS)))
			ifeq (,$(filter -DGRID,$(CXXFLAGS)))
				$(error Grid (-GRID) is required for subvariant displaced grid)
			endif
		endif
		ifneq (,$(filter -DLOOP,$(CXXFLAGS)))
			ifeq (,$(filter -DCRAZYHOUSE,$(CXXFLAGS)))
				$(error Crazyhouse (-DCRAZYHOUSE) is required for subvariant loop chess)
			endif
		endif
		ifneq (,$(filter -DPLACEMENT,$(CXXFLAGS)))
			ifeq (,$(filter -DCRAZYHOUSE,$(CXXFLAGS)))
				$(error Crazyhouse (-DCRAZYHOUSE) is required for subvariant placement chess)
			endif
		endif
		ifneq (,$(filter -DSLIPPEDGRID,$(CXXFLAGS)))
			ifeq (,$(filter -DGRID,$(CXXFLAGS)))
				$(error Grid (-GRID) is required for subvariant slipped grid)
			endif
		endif
		ifneq (,$(filter -DSUICIDE,$(CXXFLAGS)))
			ifeq (,$(filter -DANTI,$(CXXFLAGS)))
				$(error Giveaway (-DANTI) is required for subvariant suicide chess)
			endif
		endif
	endif
endif

ifeq ($(findstring USELONGESTPV,$(VARIANTS)), USELONGESTPV)
	CXXFLAGS += -DUSELONGESTPV
endif

ifeq ($(SKILL),1)
	CXXFLAGS += -DSKILL
endif

ifeq ($(debug),no)
	CXXFLAGS += -DNDEBUG
else
	CXXFLAGS += -g
endif

### 3.2.2 Debugging with undefined behavior sanitizers
ifneq ($(sanitize),no)
        CXXFLAGS += -g3 -fsanitize=$(sanitize) -fuse-ld=gold
        LDFLAGS += -fsanitize=$(sanitize) -fuse-ld=gold
endif

### 3.3 Optimization
ifeq ($(DEBUGJS),1)
	CXXFLAGS += -s ASSERTIONS=2 -s SAFE_HEAP=1 -s DEMANGLE_SUPPORT=1
	LDFLAGS += -s ASSERTIONS=2 -s SAFE_HEAP=1 -s DEMANGLE_SUPPORT=1
else
ifeq ($(optimize),yes)

	CXXFLAGS += -O3
	LDFLAGS += -O3

	ifeq ($(COMP),emscripten)
		LDFLAGS += -O3
	endif

	ifeq ($(comp),gcc)
		ifeq ($(OS), Android)
			CXXFLAGS += -fno-gcse -mthumb -march=armv7-a -mfloat-abi=softfp
		endif
	endif
	
	ifeq ($(comp),$(filter $(comp),gcc clang icc))
		ifeq ($(KERNEL),Darwin)
			CXXFLAGS += -mdynamic-no-pic
		endif
	endif
endif
endif # debugjs

### 3.4 Bits
ifeq ($(bits),64)
	CXXFLAGS += -DIS_64BIT
endif

### 3.5 prefetch
ifeq ($(prefetch),yes)
	ifeq ($(sse),yes)
		CXXFLAGS += -msse
		DEPENDFLAGS += -msse
	endif
else
	CXXFLAGS += -DNO_PREFETCH
endif

### 3.6 popcnt
ifeq ($(popcnt),yes)
	ifeq ($(comp),icc)
		CXXFLAGS += -msse3 -DUSE_POPCNT
	else
		CXXFLAGS += -msse3 -mpopcnt -DUSE_POPCNT
	endif
endif

### 3.7 pext
ifeq ($(pext),yes)
	CXXFLAGS += -DUSE_PEXT
	ifeq ($(comp),$(filter $(comp),gcc clang mingw))
		CXXFLAGS += -mbmi2
	endif
endif

### 3.8 Link Time Optimization, it works since gcc 4.5 but not on mingw under Windows.
### This is a mix of compile and link time options because the lto link phase
### needs access to the optimization flags.
ifeq ($(optimize),yes)
ifeq ($(debug), no)
	ifeq ($(comp),$(filter $(comp),gcc clang))
		CXXFLAGS += -flto
		LDFLAGS += $(CXXFLAGS)
	endif

	ifeq ($(comp),mingw)
	ifeq ($(KERNEL),Linux)
		CXXFLAGS += -flto
		LDFLAGS += $(CXXFLAGS)
	endif
	endif
endif
endif

### 3.9 Android 5 can only run position independent executables. Note that this
### breaks Android 4.0 and earlier.
ifeq ($(OS), Android)
	CXXFLAGS += -fPIE
	LDFLAGS += -fPIE -pie
endif


### ==========================================================================
### Section 4. Public targets
### ==========================================================================

help:
	@echo ""
	@echo "To compile stockfish, type: "
	@echo ""
	@echo "make target ARCH=arch [COMP=compiler] [COMPCXX=cxx]"
	@echo ""
	@echo "Supported targets:"
	@echo ""
	@echo "build                   > Standard build"
	@echo "profile-build           > PGO build"
	@echo "strip                   > Strip executable"
	@echo "install                 > Install executable"
	@echo "clean                   > Clean up"
	@echo ""
	@echo "Supported archs:"
	@echo ""
	@echo "x86-64                  > x86 64-bit"
	@echo "x86-64-modern           > x86 64-bit with popcnt support"
	@echo "x86-64-bmi2             > x86 64-bit with pext support"
	@echo "x86-32                  > x86 32-bit with SSE support"
	@echo "x86-32-old              > x86 32-bit fall back for old hardware"
	@echo "ppc-64                  > PPC 64-bit"
	@echo "ppc-32                  > PPC 32-bit"
	@echo "armv7                   > ARMv7 32-bit"
	@echo "general-64              > unspecified 64-bit"
	@echo "general-32              > unspecified 32-bit"
	@echo "js                      > emscripten javascript"
	@echo "wasm                    > emscripten webassembly"
	@echo ""
	@echo "Supported compilers:"
	@echo ""
	@echo "gcc                     > Gnu compiler (default)"
	@echo "mingw                   > Gnu compiler with MinGW under Windows"
	@echo "clang                   > LLVM Clang compiler"
	@echo "icc                     > Intel compiler"
	@echo ""
	@echo "Simple examples. If you don't know what to do, you likely want to run: "
	@echo ""
	@echo "make build ARCH=x86-64    (This is for 64-bit systems)"
	@echo "make build ARCH=x86-32    (This is for 32-bit systems)"
	@echo ""
	@echo "Advanced examples, for experienced users: "
	@echo ""
	@echo "make build ARCH=x86-64 COMP=clang"
	@echo "make profile-build ARCH=x86-64-modern COMP=gcc COMPCXX=g++-4.8"
	@echo ""


.PHONY: help build profile-build strip install clean objclean profileclean help \
        config-sanity icc-profile-use icc-profile-make gcc-profile-use gcc-profile-make \
        clang-profile-use clang-profile-make

build: config-sanity
	$(MAKE) ARCH=$(ARCH) COMP=$(COMP) all

profile-build: config-sanity objclean profileclean
	@echo ""
	@echo "Step 1/4. Building instrumented executable ..."
	$(MAKE) ARCH=$(ARCH) COMP=$(COMP) $(profile_make)
	@echo ""
	@echo "Step 2/4. Running benchmark for pgo-build ..."
	$(PGOBENCH) > /dev/null
	@echo ""
	@echo "Step 3/4. Building optimized executable ..."
	$(MAKE) ARCH=$(ARCH) COMP=$(COMP) objclean
	$(MAKE) ARCH=$(ARCH) COMP=$(COMP) $(profile_use)
	@echo ""
	@echo "Step 4/4. Deleting profile data ..."
	$(MAKE) ARCH=$(ARCH) COMP=$(COMP) profileclean

strip:
	strip $(EXE)

install:
	-mkdir -p -m 755 $(BINDIR)
	-cp $(EXE) $(BINDIR)
	-strip $(BINDIR)/$(EXE)

#clean all
clean: objclean profileclean
	@rm -f .depend *~ core

# clean binaries and objects
objclean:
	@rm -f $(EXE) stockfish.js stockfish.asm.js stockfish.wasm *.o ./syzygy/*.o

# clean auxiliary profiling files
profileclean:
	@rm -rf profdir
	@rm -f bench.txt *.gcda ./syzygy/*.gcda *.gcno ./syzygy/*.gcno
	@rm -f stockfish.profdata *.profraw

default:
	help

### ==========================================================================
### Section 5. Private targets
### ==========================================================================

all: $(EXE) .depend

config-sanity:
	@echo ""
	@echo "Config:"
	@echo "debug: '$(debug)'"
	@echo "sanitize: '$(sanitize)'"
	@echo "optimize: '$(optimize)'"
	@echo "arch: '$(arch)'"
	@echo "bits: '$(bits)'"
	@echo "kernel: '$(KERNEL)'"
	@echo "os: '$(OS)'"
	@echo "prefetch: '$(prefetch)'"
	@echo "popcnt: '$(popcnt)'"
	@echo "sse: '$(sse)'"
	@echo "pext: '$(pext)'"
	@echo ""
	@echo "Flags:"
	@echo "CXX: $(CXX)"
	@echo "CXXFLAGS: $(CXXFLAGS)"
	@echo "LDFLAGS: $(LDFLAGS)"
	@echo ""
	@echo "Testing config sanity. If this fails, try 'make help' ..."
	@echo ""
	@test "$(debug)" = "yes" || test "$(debug)" = "no"
	@test "$(sanitize)" = "undefined" || test "$(sanitize)" = "thread" || test "$(sanitize)" = "no"
	@test "$(optimize)" = "yes" || test "$(optimize)" = "no"
	@test "$(arch)" = "any" || test "$(arch)" = "x86_64" || test "$(arch)" = "i386" || \
	 test "$(arch)" = "ppc64" || test "$(arch)" = "ppc" || test "$(arch)" = "armv7"
	@test "$(bits)" = "32" || test "$(bits)" = "64"
	@test "$(prefetch)" = "yes" || test "$(prefetch)" = "no"
	@test "$(popcnt)" = "yes" || test "$(popcnt)" = "no"
	@test "$(sse)" = "yes" || test "$(sse)" = "no"
	@test "$(pext)" = "yes" || test "$(pext)" = "no"
	@test "$(comp)" = "gcc" || test "$(comp)" = "icc" || test "$(comp)" = "mingw" || test "$(comp)" = "clang"

$(EXE): $(OBJS) pre.js
	$(CXX) -o $@ $(OBJS) $(LDFLAGS)

clang-profile-make:
	$(MAKE) ARCH=$(ARCH) COMP=$(COMP) \
	EXTRACXXFLAGS='-fprofile-instr-generate ' \
	EXTRALDFLAGS=' -fprofile-instr-generate' \
	all

clang-profile-use:
	llvm-profdata merge -output=stockfish.profdata *.profraw
	$(MAKE) ARCH=$(ARCH) COMP=$(COMP) \
	EXTRACXXFLAGS='-fprofile-instr-use=stockfish.profdata' \
	EXTRALDFLAGS='-fprofile-use ' \
	all

gcc-profile-make:
	$(MAKE) ARCH=$(ARCH) COMP=$(COMP) \
	EXTRACXXFLAGS='-fprofile-generate' \
	EXTRALDFLAGS='-lgcov' \
	all

gcc-profile-use:
	$(MAKE) ARCH=$(ARCH) COMP=$(COMP) \
	EXTRACXXFLAGS='-fprofile-use -fno-peel-loops -fno-tracer' \
	EXTRALDFLAGS='-lgcov' \
	all

icc-profile-make:
	@mkdir -p profdir
	$(MAKE) ARCH=$(ARCH) COMP=$(COMP) \
	EXTRACXXFLAGS='-prof-gen=srcpos -prof_dir ./profdir' \
	all

icc-profile-use:
	$(MAKE) ARCH=$(ARCH) COMP=$(COMP) \
	EXTRACXXFLAGS='-prof_use -prof_dir ./profdir' \
	all

.depend:
	-@$(CXX) $(DEPENDFLAGS) -MM $(OBJS:.o=.cpp) > $@ 2> /dev/null

-include .depend

