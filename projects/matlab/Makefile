# DARWIN PROJECT MAKEFILE
# Stephen Gould <stephen.gould@anu.edu.au>

DRWN_PATH := $(shell pwd)/../..
PROJ_PATH := ${DRWN_PATH}/projects/matlab

#######################################################################
# define project parameters here
#######################################################################

PROJ_NAME = drwnMatlab
PROJ_MAJORVER = 1
PROJ_MINORVER = 0.0

-include $(DRWN_PATH)/make.mk

#ifeq ($(DRWN_BUILD_WITH_ZLIB), 1)
#  EXTLIB := ${EXTLIB} $(EXT_PATH)/zlib/libz.a
#endif

# matlab
ifeq ($(wildcard $(EXT_PATH)/matlab), $(EXT_PATH)/matlab)
  MEXFLAGS = CXXFLAGS='-fPIC $(PLAT_CFLAGS) $(EXTRA_CFLAGS)' \
	LDCXXFLAGS='-lm -lpthread $(PLAT_LFLAGS) $(EXTLIB) $(LIBDRWN) $(EXTRA_LFLAGS)'
  MEXEXT := $(shell $(EXT_PATH)/matlab/bin/mexext)

  ifeq ($(DRWN_BUILD_VISION_LIB), 1)
    MEXFLAGS += $(shell pkg-config --libs ${EXT_PATH}/opencv/lib/pkgconfig/opencv.pc)
    LIBDRWN += $(shell pkg-config --libs ${EXT_PATH}/opencv/lib/pkgconfig/opencv.pc)
  endif
endif

#######################################################################
# add project source files here
#######################################################################

APP_SRC = mexAnalyseClassifier.cpp mexDarwinTest.cpp mexEvalClassifier.cpp \
	mexFactorGraphInference.cpp mexKMeans.cpp mexLearnClassifier.cpp \
	mexGetLinearTransform.cpp mexSetLinearTransform.cpp \
	mexMaxFlow.cpp mexQPSolver.cpp

ifeq ($(DRWN_BUILD_VISION_LIB), 1)
  APP_SRC += mexImageCRF.cpp mexLoadSuperpixels.cpp mexSaveSuperpixels.cpp
endif

LIB_SRC =

#######################################################################

APP_PROG_NAMES = $(APP_SRC:.cpp=.${MEXEXT})
APP_OBJ = $(APP_SRC:.cpp=.o)
LIB_OBJ = $(LIB_SRC:.cpp=.o)
PROJ_DEP = ${APP_OBJ:.o=.d} ${LIB_OBJ:.o=.d}

LIB_VER = ${PROJ_MAJORVER}.${PROJ_MINORVER}

ifeq (,${LIB_SRC})
  LIB_NAMES =
  PROJLIB =
else
  LIB_NAMES = lib${PROJ_NAME}.a lib${PROJ_NAME}.so.${LIB_VER}

  ifeq ($(DRWN_SHARED_LIBS), 0)
    PROJLIB = $(BIN_PATH)/lib${PROJ_NAME}.a
  else
    PROJLIB = $(BIN_PATH)/lib${PROJ_NAME}.so.${LIB_VER}
    LFLAGS += -l${PROJ_NAME}
  endif
endif

.PHONY: clean lib app
.PRECIOUS: $(APP_OBJ) $(LIB_OBJ)

ifeq ($(wildcard $(EXT_PATH)/matlab), $(EXT_PATH)/matlab)
all: lib app
else
all:
endif

lib: ${addprefix ${BIN_PATH}/,$(LIB_NAMES)}

# applications

#app: ${addprefix ${BIN_PATH}/,$(APP_PROG_NAMES)}
#
#$(BIN_PATH)/%.${MEXEXT}: %.cpp $(PROJLIB) $(LIBDRWN)
#	$(EXT_PATH)/matlab/bin/mex -v -g $*.cpp -output $(@:.cpp=) $(PROJLIB) $(MEXFLAGS) $(LIBDRWN) $(EXTLIB)

app: 
	for i in $(APP_SRC:.cpp=); do \
	 $(EXT_PATH)/matlab/bin/mex -v -g $${i}.cpp -output $(BIN_PATH)/$${i}.${MEXEXT} $(PROJLIB) $(MEXFLAGS) $(LIBDRWN) $(EXTLIB); \
	done

# static libraries
$(BIN_PATH)/lib${PROJ_NAME}.a: $(LIB_OBJ)
	ar rcs $@ $(LIB_OBJ)

# shared libraries
$(BIN_PATH)/lib${PROJ_NAME}.so.${LIB_VER}: $(LIB_OBJ)
	${CCC} ${CFLAGS} -shared -Wl,-soname,lib${PROJ_NAME}.so.${PROJ_MAJORVER} -o $(BIN_PATH)/lib${PROJ_NAME}.so.${LIB_VER} $(LIB_OBJ)
	${LDCONFIG} -n $(BIN_PATH)

# darwin libraries
$(LIBDRWN):
	@echo "** YOU NEED TO MAKE THE DARWIN LIBRARIES FIRST **"
	false

# dependencies and object files
%.o : %.cpp
	$(CCC) -MM -MF $(subst .o,.d,$@) -MP -MT $@ $(CFLAGS) $<
	${CCC} ${CFLAGS} -c $< -o $@

# clean
clean:
	-rm $(APP_OBJ)
	-rm $(LIB_OBJ)
	-rm ${addprefix ${addprefix ${BIN_PATH}/,$(APP_PROG_NAMES)}, ".mexglx"}
	-rm ${addprefix ${addprefix ${BIN_PATH}/,$(APP_PROG_NAMES)}, ".mexa64"}
	-rm ${addprefix ${BIN_PATH}/,$(LIB_NAMES)}
	-rm ${PROJ_DEP}
	-rm *~

-include ${PROJ_DEP}
