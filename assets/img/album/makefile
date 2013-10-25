# dirs to exclude
exclude_dirs := include bin

# find subfolders
dirs := $(shell find . -maxdepth 1 -type d)
dirs := $(basename $(patsubst ./%,%,$(dirs)))
dirs := $(filter-out $(exclude_dirs),$(dirs))

.PHONY: all clean

all: $(dirs)
	for dir in $(dirs);do\
		if [ -f $$dir/Makefile ];then $(MAKE) -C $$dir;fi;\
	done

clean: 
	for dir in $(dirs);do\
		if [ -f $$dir/Makefile ];then $(MAKE) clean -C $$dir;fi;\
	done
	
