# get all jpg|png files
items := $(shell ls *.* | grep -E '*.[j][pP][gG]|*.[J][pP][gG]')
files := $(addprefix file/,$(items))
thumbs := $(addprefix thumb/,$(items))

all: $(files) $(thumbs) list.yml

# generate files
$(files): file/%: %
	[ ! -d 'file' ] && mkdir file;\
	convert -resize 1024x768 $< $@
	#convert -resize 1024x768 -gravity center -extent 1024x768 -background "#ffffff" $< $@

# generate thumbs
$(thumbs): thumb/%: %
	[ ! -d 'thumb' ] && mkdir thumb;\
	convert -resize 240x160 $< $@
	#convert -resize 240x160 -gravity center -extent 240x160 -background "#ffffff" $< $@

list.yml: $(items)
	echo 'imgs:' > $@;\
		ls file/ | sed 's/\t/\n/g' | sed 's/^/\t- /g' >> $@;\
	cat $@

.PHONY: clean

clean:
	-rm -rf file thumb list.yml

