# -- Makefile for handling TLDP documentation
#
#

default: help

DESTDIR    := output
NODESTDIR  := $(shell stat 2>/dev/null -t $(DESTDIR))
ifeq ($(NODESTDIR),)
  $(error ENOENT (2): $(DESTDIR); please create or specify alternate DESTDIR directory)
endif

WORKING    := working
DOCBASE    := $(WORKING)/ldp
NOWORKING  := $(shell stat 2>/dev/null -t $(WORKING))
ifeq ($(NOWORKING),)
  $(error ENOENT (2): $(WORKING); please create or specify alternate WORKING directory)
endif

DOCBASE    := $(abspath $(DOCBASE))

VCSBASE    := /home/mabrown/vcs/LDP/LDP
LDPDSL     := $(VCSBASE)/builder/dsssl/ldp.dsl

#OBJS       := $(VCSBASE)/faq/docbook/LDP-FAQ.xml $(VCSBASE)/howto/docbook/Traffic-Control-HOWTO/Traffic-Control-HOWTO.xml
OBJ         = $(VCSBASE)/howto/docbook/Aviation-HOWTO.xml
OBJDIR      = $(dir $(OBJ))
OBJFORMAT   = $(lastword $(subst ., ,$(suffix $(OBJ))))
OBJFILE     = $(notdir $(OBJ))
OBJSTEM     = $(OBJFILE:.$(OBJFORMAT)=)

OUTDIR      = $(DOCBASE)/$(OBJSTEM)

PDF         = $(OUTDIR)/$(OBJSTEM).pdf
HTML        = $(OUTDIR)/$(OBJSTEM).html
HTMLS       = $(OUTDIR)/$(OBJSTEM)-single.html
TEXT        = $(OUTDIR)/$(OBJSTEM).txt

all: clear_$(WORKING) $(HTMLS) $(TEXT) $(PDF) $(HTML)
	tree $(DOCBASE)
	rsync --archive --verbose --partial $(WORKING)/ $(DESTDIR)/

clear_$(WORKING):
	(test ! -d $(DOCBASE) || ( cd $(WORKING) && rm -rf -- $(notdir $(DOCBASE))))

$(DOCBASE): $(DESTDIR)
	mkdir -p $(DOCBASE)

$(OUTDIR): $(DOCBASE)
	mkdir -p $(OUTDIR)

$(OUTDIR)/images: $(OUTDIR)
	(cd $(OBJDIR) && test ! -d images || cp -rva images $(OUTDIR))

$(HTMLS): $(OUTDIR) $(OUTDIR)/images
	# -- note the mv -vu $(notdir $(HTML)) $(notdir $(HTMLS))
	#    the docbook2html processor will create a single-page
	#    HTML file called $(OBJSTEM).html, which we will want
	#    to create as a symlink, later
	(cd $(OUTDIR) \
          && jw -f docbook -b html \
	      --dsl "$(LDPDSL)#html" \
	      -V nochunks \
	      -V '%callout-graphics-path%=images/callouts/' \
	      -V '%stock-graphics-extension%=.png' \
	      -V '%stylsheet-type%=freddie.css' \
	      --output . \
	      $(OBJ) \
          && mv -vu $(notdir $(HTML)) $(notdir $(HTMLS)))
	# -- what about images and other resources?

$(TEXT): $(HTMLS)
	(cd $(OUTDIR) && html2text -style pretty -nobs $(notdir $(HTMLS)) > $(notdir $@))

$(PDF): $(OUTDIR)
	( cd $(OUTDIR) \
          && jw -f docbook -b pdf --output . $(OBJ))

$(HTML): $(OUTDIR)
	# -- the jade DocBook processing toolchain produces an HTML output
	#    file called index.html in the chunked output; therefore, we want
	#    to create a link from Some-Name.html to index.html.
	( cd $(OUTDIR) \
          && jw -f docbook -b html  \
	      --dsl "$(LDPDSL)#html" \
	      -V '%admon-graphics-path%=images/' \
	      -V '%callout-graphics-path%=images/callouts/' \
	      -V '%stock-graphics-extension%=.png' \
	      --output . \
	      $(OBJ) \
          && ln -snvf index.html $(notdir $(HTML)))
	# -- what about images and other resources?

vars:
	printf "%s\n" \
	  "OBJ     = $(OBJ)" \
	  "OBJFILE = $(OBJFILE)" \
	  "OBJSTEM = $(OBJSTEM)" \
	  "DOCS    = $(DOCS)" \
	  "PDF     = $(PDF)" \
	  "HTML    = $(HTML)" \
	  "HTMLS   = $(HTMLS)" \
	  "TEXT    = $(TEXT)" \



.PHONY: help
help:
	@printf "%s\n" \
	"There will be help here in the future."

#
# -- end of file
