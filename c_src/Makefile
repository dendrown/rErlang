# Based on c_src.mk from erlang.mk by Loic Hoguin <essen@ninenines.eu>

VERSION := 0.1
CURDIR := $(shell pwd)
BASEDIR := $(abspath $(CURDIR)/..)

PROJECT ?= $(notdir $(BASEDIR))
PROJECT := $(strip $(PROJECT))

R_HOME ?= $(shell R RHOME)
R_INCLUDE_DIR ?= $(R_HOME)/include
R_LIB_DIR ?= $(R_HOME)/lib
ERTS_INCLUDE_DIR ?= $(shell erl -noshell -s init stop -eval "io:format(\"~s/erts-~s/include/\", [code:root_dir(), erlang:system_info(version)]).")
ERL_INTERFACE_INCLUDE_DIR ?= $(shell erl -noshell -s init stop -eval "io:format(\"~s\", [code:lib_dir(erl_interface, include)]).")
ERL_INTERFACE_LIB_DIR ?= $(shell erl -noshell -s init stop -eval "io:format(\"~s\", [code:lib_dir(erl_interface, lib)]).")

C_SRC_DIR = $(CURDIR)
PRODUCT_ERI ?= $(CURDIR)/../priv/ERI-$(VERSION)
PRODUCT_RE ?= $(CURDIR)/../priv/rErlang-$(VERSION)

# System type and C compiler/flags.

UNAME_SYS := $(shell uname -s)
ifeq ($(UNAME_SYS), Darwin)
	CC ?= cc
	CFLAGS ?= -O2 -std=gnu99 -arch x86_64 -finline-functions -Wall -Wmissing-prototypes
	CXXFLAGS ?= -O2 -arch x86_64 -finline-functions -Wall
	LDFLAGS ?= -arch x86_64 -flat_namespace -undefined suppress
else ifeq ($(UNAME_SYS), FreeBSD)
	CC ?= cc
	CFLAGS ?= -O2 -std=gnu99 -finline-functions -Wall -Wmissing-prototypes
	CXXFLAGS ?= -O2 -finline-functions -Wall
else ifeq ($(UNAME_SYS), Linux)
	CC ?= gcc
	CFLAGS ?= -O2 -std=gnu99 -finline-functions -Wall -Wmissing-prototypes
	CXXFLAGS ?= -O2 -finline-functions -Wall
endif

CFLAGS += -fPIC -I $(ERTS_INCLUDE_DIR) -I $(ERL_INTERFACE_INCLUDE_DIR) -I $(R_INCLUDE_DIR)
CXXFLAGS += -fPIC -I $(ERTS_INCLUDE_DIR) -I $(ERL_INTERFACE_INCLUDE_DIR) -I $(R_INCLUDE_DIR)

LDLIBS += -L $(ERL_INTERFACE_LIB_DIR) -L $(R_LIB_DIR) -lR -lerl_interface_st -lei_st -lerl_interface -lei
LDFLAGS += 

# Verbosity.

c_verbose_0 = @echo " C     " $(?F);
c_verbose = $(c_verbose_$(V))

cpp_verbose_0 = @echo " CPP   " $(?F);
cpp_verbose = $(cpp_verbose_$(V))

link_verbose_0 = @echo " LD    " $(@F);
link_verbose = $(link_verbose_$(V))

SOURCES := $(shell find $(C_SRC_DIR) -type f \( -name "*.c" -o -name "*.C" -o -name "*.cc" -o -name "*.cpp" \))
OBJECTS = $(addsuffix .o, $(basename $(SOURCES)))

COMPILE_C = $(c_verbose) $(CC) $(CFLAGS) $(CPPFLAGS) -c
COMPILE_CPP = $(cpp_verbose) $(CXX) $(CXXFLAGS) $(CPPFLAGS) -c

all: $(PRODUCT_ERI) $(PRODUCT_RE).so
$(PRODUCT_ERI): $(OBJECTS)
	@mkdir -p $(BASEDIR)/priv/
	$(link_verbose) $(CC) $(OBJECTS) $(LDFLAGS) $(LDLIBS) -o $(PRODUCT_ERI)

$(PRODUCT_RE).so: $(OBJECTS)
	$(COMPILE_C) -fpic -g -c rErlang.c -o $(PRODUCT_RE).o
	$(link_verbose) $(CC) ${PRODUCT_RE}.o -shared $(LDFLAGS) $(LDLIBS) -o $(PRODUCT_RE).so

%.o: %.c
	$(COMPILE_C) $(OUTPUT_OPTION) $<

%.o: %.cc
	$(COMPILE_CPP) $(OUTPUT_OPTION) $<

%.o: %.C
	$(COMPILE_CPP) $(OUTPUT_OPTION) $<

%.o: %.cpp
	$(COMPILE_CPP) $(OUTPUT_OPTION) $<

clean:
	@rm -f $(C_SRC_OUTPUT) $(OBJECTS)
