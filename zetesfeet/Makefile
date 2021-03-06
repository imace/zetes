APPLICATION_NAME = ZetesFeet

UNAME := $(shell uname)
ifndef ARCH
  ARCH := $(shell uname -m)
endif

ifndef TARGET
  TARGET := ./target
endif

SRC = src
BIN = bin
OBJ = obj
LIB = lib

DEBUG_OPTIMIZE = -O0 -g

ifeq ($(UNAME), Darwin)	# OS X
  JAVA_HOME = $(shell /usr/libexec/java_home)
  PLATFORM_TAG = osx-x86_64
  PLATFORM_GENERAL_INCLUDES = -I"$(JAVA_HOME)/include" -I"$(JAVA_HOME)/include/darwin"
  EXE_EXT=
else ifeq ($(UNAME) $(ARCH), Linux x86_64)		# linux on PC
  PLATFORM_TAG = linux-x86_64
  PLATFORM_GENERAL_INCLUDES = -I"$(JAVA_HOME)/include" -I"$(JAVA_HOME)/include/linux"
  EXE_EXT=
else ifeq ($(UNAME) $(ARCH), Linux armv6l)		# linux on Raspberry Pi
  PLATFORM_TAG = linux-armv6l
  PLATFORM_GENERAL_INCLUDES = -I"$(JAVA_HOME)/include" -I"$(JAVA_HOME)/include/linux"
  EXE_EXT=
else ifeq ($(OS) $(ARCH), Windows_NT i686)		# Windows 32
  PLATFORM_TAG = win-i386
  PLATFORM_GENERAL_INCLUDES = -I"$(JAVA_HOME)/include" -I"$(JAVA_HOME)/include/win32"
  EXE_EXT=.exe
else ifeq ($(OS) $(ARCH), Windows_NT x86_64)	# Windows 64
  PLATFORM_TAG = win-x86_64
  PLATFORM_GENERAL_INCLUDES = -I"$(JAVA_HOME)/include" -I"$(JAVA_HOME)/include/win32"
  EXE_EXT=.exe
endif

# Java platform agnostic
JAVA_SOURCE_PATH = $(SRC)/java
JAVA_FILES = $(shell cd $(JAVA_SOURCE_PATH); find . -type f -name \*.java | awk '{ sub(/.\//,"") }; 1')
JAVA_CLASSES = $(addprefix $(JAVA_CLASSPATH)/,$(addsuffix .class,$(basename $(JAVA_FILES))))

# Java platform specific
JAVA_PLATFORM_SPECIFIC_SOURCE_PATH = $(SRC)/$(PLATFORM_TAG)/java
JAVA_PLATFORM_SPECIFIC_FILES = $(shell if [ -d "$(JAVA_PLATFORM_SPECIFIC_SOURCE_PATH)" ]; then cd $(JAVA_PLATFORM_SPECIFIC_SOURCE_PATH); find . -type f -name \*.java | awk '{ sub(/.\//,"") }; 1'; fi)
JAVA_PLATFORM_SPECIFIC_CLASSES = $(addprefix $(JAVA_CLASSPATH)/,$(addsuffix .class,$(basename $(JAVA_PLATFORM_SPECIFIC_FILES))))

# C++ Platform agnostic
CPP_SOURCE_PATH = $(SRC)/cpp
CPP_FILES = $(shell cd $(CPP_SOURCE_PATH); find . -type f -name \*.cpp | awk '{ sub(/.\//,"") }; 1')
CPP_HEADER_FILES = $(addprefix $(CPP_SOURCE_PATH)/,$(shell cd $(CPP_SOURCE_PATH); find . -type f -name \*.h | awk '{ sub(/.\//,"") }; 1'))
CPP_OBJECTS = $(addprefix $(OBJECTS_PATH)/,$(addsuffix .o,$(basename $(CPP_FILES))))

# Target paths
LIBRARY_PATH = $(TARGET)/$(LIB)/$(PLATFORM_TAG)
OBJECTS_PATH = $(TARGET)/$(OBJ)/$(PLATFORM_TAG)
JAVA_LIBRARY_PATH = $(TARGET)/$(LIB)/java
JAVA_OBJECTS_PATH = $(TARGET)/$(OBJ)/java
JAVA_CLASSPATH = $(JAVA_OBJECTS_PATH)/classes

# Target names
LIBRARY_NAME = libzetesfeet.a
JAVA_LIBRARY_NAME = zetesfeet.jar
TOOLS_PATH = tools/$(PLATFORM_TAG)
BIN_TO_OBJ = $(TOOLS_PATH)/binaryToObject$(EXE_EXT)
PACKAGE_NAME = zetesfeet.zip

help:
	@echo "*** This is Zetes build utility: This part builds feet of Zetes ***"
	@echo
	@echo 1. Set up your build environment
	@echo Zetes library needs Java 7 compiler, so JAVA_HOME is to be set correctly. 
	@echo
	@echo Current value: JAVA_HOME=$(JAVA_HOME)
	@echo
	@echo To change it you can type:
	@echo
	@echo "  > export JAVA_HOME=<path>"
	@echo
	@echo Make sure that the current value ARCH=$(ARCH) is a correct architecture of the
	@echo target that you are going to build. If it\'s not, type
	@echo
	@echo "  > export ARCH=<arch>"
	@echo
	@echo "where <arch> is one of: i386, x86_64"
	@echo
	@echo 2. Build Zetes or config IDE 
	@echo To build the library right now from the command line, type 
	@echo
	@echo "  > make all"
	@echo
	@echo To setup project files for Eclipse CDT IDE, type
	@echo
	@echo "  > make ideconf-eclipse"
	@echo
	@echo To clean compiled binary files, type 
	@echo
	@echo "  > make clean"
	@echo

# Other files that should just be copied to the target folder 
JUST_COPY_FILES = $(BIN_TO_OBJ) include/FeetStarter.h
include ../common-scripts/just_copy.mk

all: $(LIBRARY_PATH)/$(LIBRARY_NAME) $(JAVA_LIBRARY_PATH)/$(JAVA_LIBRARY_NAME) $(JUST_COPY_FILES_TARGET)
	@echo "*** $(APPLICATION_NAME) building process completed successfully. ***"
	@echo You can find the result in folders:
	@echo
	@echo "  $(realpath $(LIBRARY_PATH))"
	@echo "  $(realpath $(JAVA_LIBRARY_PATH))"
	@echo

package: $(TARGET)/$(PACKAGE_NAME)

include ../common-scripts/ideconf/ideconf.mk

$(JAVA_CLASSPATH)/%.class: $(JAVA_SOURCE_PATH)/%.java
	@echo [$(APPLICATION_NAME)] Compiling $<...
	if [ ! -d "$(dir $@)" ]; then mkdir -p "$(dir $@)"; fi
	"$(JAVA_HOME)/bin/javac" -encoding utf8 -sourcepath "$(JAVA_SOURCE_PATH)" -classpath "$(JAVA_CLASSPATH)" -d "$(JAVA_CLASSPATH)" $<

$(JAVA_CLASSPATH)/%.class: $(JAVA_PLATFORM_SPECIFIC_SOURCE_PATH)/%.java
	@echo [$(APPLICATION_NAME)] Compiling platform specific $<...
	if [ ! -d "$(dir $@)" ]; then mkdir -p "$(dir $@)"; fi
	"$(JAVA_HOME)/bin/javac" -encoding utf8 -sourcepath "$(JAVA_PLATFORM_SPECIFIC_SOURCE_PATH)" -classpath "$(JAVA_CLASSPATH)" -d "$(JAVA_CLASSPATH)" $<

$(OBJECTS_PATH)/%.o: $(SRC)/cpp/%.cpp $(CPP_HEADER_FILES)
	@echo [$(APPLICATION_NAME)] Compiling $<...
	mkdir -p $(OBJECTS_PATH)
	g++ $(DEBUG_OPTIMIZE) -D_JNI_IMPLEMENTATION_ -c -Iinclude $(PLATFORM_GENERAL_INCLUDES) $< -o $@

$(LIBRARY_PATH)/$(LIBRARY_NAME): $(CPP_OBJECTS) lib/$(PLATFORM_TAG)/libavian.a
	@echo [$(APPLICATION_NAME)] Constructing $@...
	mkdir -p $(LIBRARY_PATH);
	mkdir -p $(OBJECTS_PATH);
	
	# Extracting libavian objects
	( \
	    cd $(OBJECTS_PATH); \
	    mkdir -p libavian; \
	    cd libavian; \
	    ar x $(CURDIR)/lib/$(PLATFORM_TAG)/libavian.a; \
	)
	
	# Extracting libcrypto objects
	( \
	    cd $(OBJECTS_PATH); \
	    mkdir -p libcrypto; \
	    cd libcrypto; \
	    ar x $(CURDIR)/lib/$(PLATFORM_TAG)/libcrypto.a; \
	)
	
	# Extracting libssl objects
	( \
	    cd $(OBJECTS_PATH); \
	    mkdir -p libssl; \
	    cd libssl; \
	    ar x $(CURDIR)/lib/$(PLATFORM_TAG)/libssl.a; \
	)
	
	# Extracting libexpat objects
	( \
	    cd $(OBJECTS_PATH); \
	    mkdir -p libexpat; \
	    cd libexpat; \
	    ar x $(CURDIR)/lib/$(PLATFORM_TAG)/libexpat.a; \
	)

	# Extracting libicui18n objects
	( \
	    cd $(OBJECTS_PATH); \
	    mkdir -p libicui18n; \
	    cd libicui18n; \
	    ar x $(CURDIR)/lib/$(PLATFORM_TAG)/libicui18n.a; \
	)

	# Extracting libicuuc objects
	( \
	    cd $(OBJECTS_PATH); \
	    mkdir -p libicuuc; \
	    cd libicuuc; \
	    ar x $(CURDIR)/lib/$(PLATFORM_TAG)/libicuuc.a; \
	)

	# Extracting libicudata objects
	( \
	    cd $(OBJECTS_PATH); \
	    mkdir -p libicudata; \
	    cd libicudata; \
	    ar x $(CURDIR)/lib/$(PLATFORM_TAG)/libicudata.a; \
	)
	
	# Extracting libfdm objects
	( \
	    cd $(OBJECTS_PATH); \
	    mkdir -p libfdm; \
	    cd libfdm; \
	    ar x $(CURDIR)/lib/$(PLATFORM_TAG)/libfdm.a; \
	)
	
	mkdir -p $(JAVA_LIBRARY_PATH)

	# Making an object file from the java class library
	
	ar rvs $@ $(CPP_OBJECTS)
	ar rvs $@ $(OBJECTS_PATH)/libavian/*.o
	ar rvs $@ $(OBJECTS_PATH)/libcrypto/*.o
	ar rvs $@ $(OBJECTS_PATH)/libssl/*.o
	ar rvs $@ $(OBJECTS_PATH)/libexpat/*.o
	ar rvs $@ $(OBJECTS_PATH)/libicudata/*.o
	ar rvs $@ $(OBJECTS_PATH)/libicuuc/*.ao
	ar rvs $@ $(OBJECTS_PATH)/libicui18n/*.ao
	ar rvs $@ $(OBJECTS_PATH)/libfdm/*.o

$(JAVA_LIBRARY_PATH)/$(JAVA_LIBRARY_NAME): lib/java/classpath.jar $(JAVA_CLASSES) $(JAVA_PLATFORM_SPECIFIC_CLASSES)
	@echo [$(APPLICATION_NAME)] Constructing $@...
	mkdir -p $(LIBRARY_PATH);

	# Making the java class library
	cp -f lib/java/classpath.jar $(JAVA_LIBRARY_PATH)/$(JAVA_LIBRARY_NAME); \
	( \
	    cd $(JAVA_LIBRARY_PATH); \
	    "$(JAVA_HOME)/bin/jar" uf $(JAVA_LIBRARY_NAME) -C $(CURDIR)/$(JAVA_CLASSPATH) .; \
	)

$(TARGET)/$(PACKAGE_NAME): $(LIBRARY_PATH)/$(LIBRARY_NAME) $(JAVA_LIBRARY_PATH)/$(JAVA_LIBRARY_NAME) $(JUST_COPY_FILES_TARGET)
	@echo [$(APPLICATION_NAME)] Making package file $@...
	( \
	    cd $(TARGET); \
	    zip $(PACKAGE_NAME) $(subst $(TARGET)/,,$(LIBRARY_PATH)/$(LIBRARY_NAME) $(JAVA_LIBRARY_PATH)/$(JAVA_LIBRARY_NAME) $(JUST_COPY_FILES_TARGET)); \
	)

clean:
	@echo [$(APPLICATION_NAME)] Cleaning all...
	rm -rf $(TARGET)
	

.PHONY: all package help
.SILENT:
