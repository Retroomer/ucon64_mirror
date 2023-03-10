ifeq ($(CC),cc)
CC=gcc
endif

# I think we only use gnu99 instead of c99 due to va_args extensions.
CFLAGS+=-I. -Wall -W -std=gnu99
ifneq ($(findstring clang,$(CC)),clang)
CFLAGS+=-Wno-format-truncation
endif
ifdef DEBUG
CFLAGS+=-pg -g -pedantic -ansi -DDEBUG
else
CFLAGS+=-O3 -funroll-loops
ifneq ($(findstring clang,$(CC)),clang)
CFLAGS+=-fexpensive-optimizations
endif
endif

ifndef DJGPP
# uname is not available by default under DOS
OSTYPE=$(shell uname -s)
else
OSTYPE=DJGPP
endif

GCC_WIN=0
ifeq ($(findstring CYGWIN,$(OSTYPE)),CYGWIN)
GCC_WIN=1
endif
ifeq ($(findstring MINGW,$(OSTYPE)),MINGW)
GCC_WIN=1
endif

ifneq ($(OSTYPE),DJGPP)
ifeq ($(OSTYPE),BeOS)
LDFLAGS+=-nostart
else # UNIX or Win GCC
LDFLAGS+=-shared
endif
endif

ifneq ($(OSTYPE),DJGPP)
ifneq ($(GCC_WIN),1)
CFLAGS+=-fPIC
else
# Cygwin and MinGW need an import library for a DLL
LDFLAGS+=-Wl,--out-implib,libcd64dll.a
endif
endif


DEFAULT_BUILD=1

# If the user passed anything, we are not a default build.

ifdef LIBIEEE1284
DEFAULT_BUILD=0
else
ifdef PPDEV
DEFAULT_BUILD=0
else
ifdef PORTDEV
DEFAULT_BUILD=0
else
ifdef RAWIO
DEFAULT_BUILD=0
endif
endif
endif
endif

ifeq ($(DEFAULT_BUILD),1)
# Put default build options for each OS here

ifeq ($(OSTYPE),DJGPP)
RAWIO=1
endif

ifeq ($(findstring MINGW,$(OSTYPE)),MINGW)
RAWIO=1
endif

ifeq ($(findstring CYGWIN,$(OSTYPE)),CYGWIN)
RAWIO=1
endif

ifeq ($(OSTYPE),BeOS)
RAWIO=1
endif

ifeq ($(OSTYPE),OpenBSD)
RAWIO=1
endif

ifeq ($(OSTYPE),NetBSD)
RAWIO=1
endif

ifeq ($(OSTYPE),FreeBSD)
RAWIO=1
endif

ifeq ($(OSTYPE),Linux)
#ifeq ($(shell if test -r /usr/include/ieee1284.h; then echo 1; else echo 0; fi),1)
#LIBIEEE1284=1
#endif
ifeq ($(shell if test -r /usr/include/linux/ppdev.h; then echo 1; else echo 0; fi),1)
PPDEV=1
endif
#PORTDEV=1
#RAWIO=1
endif

endif # DEFAULT_BUILD = 1

# Now for backend-specific defs

ifdef LIBIEEE1284
CFLAGS+=-DCD64_USE_LIBIEEE1284
LIBS+=/usr/lib/libieee1284.a
LDFLAGS+=-lieee1284
endif

ifdef PPDEV
CFLAGS+=-DCD64_USE_PPDEV
endif

ifdef PORTDEV
CFLAGS+=-DCD64_USE_PORTDEV
endif

ifdef RAWIO
CFLAGS+=-DCD64_USE_RAWIO

ifeq ($(OSTYPE),OpenBSD)
ifeq ($(shell if test -r /usr/lib/libamd64.a; then echo 1; else echo 0; fi),1)
LIBS+=/usr/lib/libamd64.a
LDFLAGS+=-lamd64
else
ifeq ($(shell if test -r /usr/lib/libi386.a; then echo 1; else echo 0; fi),1)
LIBS+=/usr/lib/libi386.a
LDFLAGS+=-li386
endif
endif
endif # OpenBSD

ifeq ($(OSTYPE),NetBSD)
ifeq ($(shell if test -r /usr/lib/libx86_64.a; then echo 1; else echo 0; fi),1)
LIBS+=/usr/lib/libx86_64.a
LDFLAGS+=-lx86_64
else
ifeq ($(shell if test -r /usr/lib/libi386.a; then echo 1; else echo 0; fi),1)
LIBS+=/usr/lib/libi386.a
LDFLAGS+=-li386
endif
endif
endif # NetBSD
endif # RAWIO

default: all

ifeq ($(OSTYPE),DJGPP)
all: libcd64.a
else
ifeq ($(GCC_WIN),1)
all: libcd64.a cd64.dll
else
all: libcd64.a libcd64.so
endif # GCC_WIN
endif # DJGPP

# libcd64 stuff

.c.o:
	$(CC) $(CFLAGS) -c $< -o $@

libcd64.a: cd64lib.o cd64io.o
	$(LD) -r $^ $(LIBS) -o $*.o
	$(AR) crs $@ $*.o

ifeq ($(GCC_WIN),1)
cd64.dll: cd64lib.o cd64io.o
else
libcd64.so: cd64lib.o cd64io.o
endif
	$(CC) $^ $(LDFLAGS) -o $@

clean:
	rm -f *.o *.so *.dll *.a a.out
