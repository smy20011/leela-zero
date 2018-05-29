THE_OS := $(shell uname -s)

default:
	@echo "Detected OS: ${THE_OS}"
	$(MAKE) CC=gcc CXX=g++ \
		CXXFLAGS='$(CXXFLAGS) -Wall -Wextra -pipe -O3 -g -ffast-math -flto -march=native -std=c++14 -DNDEBUG'  \
		LDFLAGS='$(LDFLAGS) -flto -g' \
		leelaz

debug:
	@echo "Detected OS: ${THE_OS}"
	$(MAKE) CC=gcc CXX=g++ \
		CXXFLAGS='$(CXXFLAGS) -Wall -Wextra -pipe -Og -g -std=c++14' \
		LDFLAGS='$(LDFLAGS) -g' \
		leelaz

clang:
	@echo "Detected OS: ${THE_OS}"
	$(MAKE) CC=clang-5.0 CXX=clang++-5.0 \
		CXXFLAGS='$(CXXFLAGS) -Wall -Wextra -Wno-missing-braces -O3 -ffast-math -flto -march=native -std=c++14 -DNDEBUG' \
		LDFLAGS='$(LDFLAGS) -flto -fuse-linker-plugin' \
		leelaz

DYNAMIC_LIBS = -lboost_program_options -lpthread -lz -L/usr/local/lib -lgrpc++ -lprotobuf -pthread
CXXFLAGS += -pthread -I/usr/local/include
LIBS = -L . -ltfclient
CLIENT_OBJECTS = $(shell find client/ -type f -name "*.cc" -print | sed 's/cc$$/o/g')

ifeq ($(THE_OS),Linux)
# for Linux with OpenBLAS
	CXXFLAGS += -I/usr/include/openblas
	DYNAMIC_LIBS += -lopenblas
	DYNAMIC_LIBS += -lOpenCL
endif
ifeq ($(THE_OS),Darwin)
# for macOS (comment out the Linux part)
	LIBS += -framework Accelerate
	LIBS += -framework OpenCL
	CXXFLAGS += -I/System/Library/Frameworks/Accelerate.framework/Versions/Current/Headers
endif

# for MKL instead of OpenBLAS
#DYNAMIC_LIBS += -lmkl_rt
#CXXFLAGS += -I/opt/intel/mkl/include
#LDFLAGS  += -L/opt/intel/mkl/lib/intel64/

CXXFLAGS += -I. -I client
CPPFLAGS += -MD -MP

sources = Network.cpp FullBoard.cpp KoState.cpp Training.cpp \
	  TimeControl.cpp UCTSearch.cpp GameState.cpp Leela.cpp \
	  SGFParser.cpp Timing.cpp Utils.cpp FastBoard.cpp \
	  SGFTree.cpp Zobrist.cpp FastState.cpp GTP.cpp Random.cpp \
	  SMP.cpp UCTNode.cpp UCTNodePointer.cpp UCTNodeRoot.cpp \
	  OpenCL.cpp OpenCLScheduler.cpp NNCache.cpp Tuner.cpp

objects = $(sources:.cpp=.o)
deps = $(sources:%.cpp=%.d)

-include $(deps)

%.o: %.cpp
	$(CXX) $(CXXFLAGS) $(CPPFLAGS) -c -o $@ $<

%.o: %.cc
	$(CXX) $(CXXFLAGS) $(CPPFLAGS) -c -o $@ $<

libtfclient.a: $(CLIENT_OBJECTS)
	gcc-ar rcs libtfclient.a $^

leelaz: $(objects) libtfclient.a
	$(CXX) $(LDFLAGS) -o $@ $(objects) $(LIBS) $(DYNAMIC_LIBS)

proto_files:
	bash -c "protoc -I tensorflow/ -I serving/ --grpc_out=client/ --plugin=protoc-gen-grpc=`which grpc_cpp_plugin` serving/tensorflow_serving/apis/prediction_service.proto"
	bash -c "shopt -s globstar; protoc -I tensorflow/ -I serving/ --cpp_out=client/ serving/**/*.proto"
	bash -c "shopt -s globstar; protoc -I tensorflow/ -I serving/ --cpp_out=client/ tensorflow/tensorflow/core/**/*.proto"

clean:
	-$(RM) leelaz $(objects) $(deps)

print-%  : ; @echo $* = $($*)
.PHONY: clean default debug clang proto_files
