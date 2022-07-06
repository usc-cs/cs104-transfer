CXX=g++
CXXFLAGS=-g -Wall -std=c++11 
GTESTINCL := -I /usr/include/gtest/  
GTESTLIBS := -lgtest -lgtest_main  -lpthread
# Uncomment for parser DEBUG
#DEFS=-DDEBUG

.PHONY: clean 

all: ht-test 

ht-test: ht-test.cpp ht.h
	$(CXX) $(CXXFLAGS) $(DEFS) $< -o $@

clean:
	rm -f *~ *.o ht-test 
