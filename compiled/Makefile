
# location of the Boost Python include files and library
PYTHON_INCLUDE = `python3-config --includes`
PYTHON_LIB=`python3-config --libs`


LINUXADD= #-Wl --export-dynamic
ROOTSTUFF=`root-config --libs --glibs --ldflags`
ROOTCFLAGS=`root-config  --cflags`
CPP_FILES := $(wildcard src/*.cpp)
OBJ_FILES := $(addprefix obj/,$(notdir $(CPP_FILES:.cpp=.o)))

BINS := $(patsubst bin/%.cpp, %, $(wildcard bin/*.cpp))

MODULES := $(wildcard src/*.C)
MODULES_OBJ_FILES := $(addprefix ./,$(notdir $(MODULES:.C=.o)))
MODULES_SHARED_LIBS := $(addprefix ./,$(notdir $(MODULES:.C=.so)))

#CFLAGS := -g

all: $(MODULES_SHARED_LIBS) $(patsubst to_bin/%.cpp, %, $(wildcard to_bin/*.cpp)) classdict.so

classdict.cxx: src/LinkDef.h
	rootcling -v4 -f $@  -rmf classdict.rootmap -rml classdict.so  $^

classdict.so: classdict.cxx
	g++ -o classdict.so classdict.cxx $(CFLAGS) $(ROOTCFLAGS) $(ROOTSTUFF) -std=c++11 -fPIC -shared 	


%: to_bin/%.cpp libdeepjetcorehelpers.so classdict.cxx
	g++ -o $@ $(CFLAGS) -I./interface $(LINUXADD) $(PYTHON_INCLUDE) $< -L. -ldeepjetcorehelpers -lquicklz $(PYTHON_LIB) -lboost_python3 -lboost_numpy3 $(ROOTCFLAGS) $(ROOTSTUFF)    
	mv $@ ../bin/

#helpers
libquicklz.so:
	gcc -o libquicklz.so -shared -O2 -fPIC src/quicklz.c 

obj/%.o: src/%.cpp
	g++ -o $@ $< $(CFLAGS) $(PYTHON_INCLUDE) $(ROOTCFLAGS) -I./interface -O2 -fPIC -c  

#pack helpers in lib
libdeepjetcorehelpers.so: $(OBJ_FILES) 
	g++ -o $@ -shared -fPIC $(LINUXADD) $(CFLAGS) -fPIC  obj/*.o $(ROOTSTUFF) $(PYTHON_LIB) -lboost_python3 -lboost_numpy3 

%.so: %.o libdeepjetcorehelpers.so libquicklz.so
	g++  -o $(@) -shared -g -fPIC $(CFLAGS) $(LINUXADD) $<   $(ROOTSTUFF) -L./ -lquicklz  $(PYTHON_LIB) -lboost_python3 -lboost_numpy3 -L./ -ldeepjetcorehelpers 

%.o: src/%.C 
	g++ -fPIC -o $(@) $< $(ROOTCFLAGS) -O2 -g -I./interface $(PYTHON_INCLUDE) $(CFLAGS) -c 



clean: 
	rm -f $(OBJ_FILES) $(SHARED_LIBS) $(MODULES_SHARED_LIBS) $(MODULES_OBJ_FILES) libdeepjetcorehelpers.so libquicklz.so classdict.so classdict.cxx classdict.rootmap classdict_rdict.pcm
