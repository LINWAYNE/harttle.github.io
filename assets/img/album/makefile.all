# dirs to exclude
exclude_dirs := include bin

# find subfolders
dirs := $(shell find . -maxdepth 1 -type d)
dirs := $(basename $(patsubst ./%,%,$(dirs)))
dirs := $(filter-out $(exclude_dirs),$(dirs))

.PHONY: all clean-all

all: $(dirs)
	for dir in $(dirs);do\
		$(MAKE) -C $$dir;\
	done

clean-all: 
	for dir in $(dirs);do\
		$(MAKE) clean-all -C $$dir;\
	done
