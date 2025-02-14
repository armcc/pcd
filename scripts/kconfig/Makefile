VERSION=1.4
HOST_CC=gcc
HOST_CXX=g++
HOST_CFLAGS=-O2 -Wall -Wno-unused-but-set-variable -Wno-unused-result -Wno-unused-function -g -fPIC
HOST_LDFLAGS=
LEX=flex
LFLAGS=-L
YACC=bison
YFLAGS=-l
#YFLAGS=-t -v -l
GTKCFLAGS=`pkg-config gtk+-2.0 gmodule-2.0 libglade-2.0 --cflags`
GTKLDFLAGS=`pkg-config gtk+-2.0 gmodule-2.0 libglade-2.0 --libs`

HOST_CFLAGS += -DPRODUCT_NAME=\"$(PRODUCT_NAME)\" -DPRODUCT=\"$(PRODUCT)\"

parse_SRC=zconf.y
conf_SRC=conf.c $(parse_SRC)
mconf_SRC=mconf.c $(parse_SRC)
gconf_SRC=gconf.c kconfig_load.c
HDR=expr.h lkc.h lkc_proto.h
OTHER=README lkc-language.txt Makefile.kernel \
	kconfig.i extconf.rb example
INST=zconf.y zconf.l confdata.c expr.c symbol.c menu.c \
	conf.c mconf.c gconf.c gconf.glade kconfig_load.c images.c $(HDR)
INSTGEN=lex.zconf.c zconf.tab.c

## DEBUG set to avoid dlopen on libkconfig.so
DEBUG=1
ifdef DEBUG
HOST_CFLAGS+=-DLKC_DIRECT_LINK
endif

SRC=$(conf_SRC) $(mconf_SRC)
CSRC=$(filter %.c, $(SRC))
YSRC=$(filter %.y, $(SRC))
LSRC=$(filter %.l, $(SRC))

parse_OBJ=$(filter %.o, \
		$(patsubst %.c,%.o, \
		$(patsubst %.y,%.tab.o, \
		$(patsubst %.l,lex.%.o, \
			$(parse_SRC)))))
conf_OBJ=$(filter %.o, \
		$(patsubst %.c,%.o, \
		$(patsubst %.y,%.tab.o, \
		$(patsubst %.l,lex.%.o, \
			$(conf_SRC)))))
mconf_OBJ=$(filter %.o, \
		$(patsubst %.c,%.o, \
		$(patsubst %.y,%.tab.o, \
		$(patsubst %.l,lex.%.o, \
			$(mconf_SRC)))))
gconf_OBJ=$(filter %.o, \
		$(patsubst %.c,%.o, \
		$(patsubst %.y,%.tab.o, \
		$(patsubst %.l,lex.%.o, \
			$(gconf_SRC)))))
OBJ=$(conf_OBJ) $(mconf_OBJ) kconfig_load.o

all: conf mconf

lkc_deps := lkc.h lkc_proto.h lkc_defs.h expr.h

zconf.tab.c: zconf.y
zconf.tab.h: zconf.y
lex.zconf.c: zconf.l
zconf.tab.o: zconf.tab.c lex.zconf.c confdata.c expr.c symbol.c menu.c $(lkc_deps)
#lex.zconf.o: lex.zconf.c zconf.tab.h $(lkc_deps)
#confdata.o: confdata.c $(lkc_deps)
#expr.o: expr.c $(lkc_deps)
#symbol.o: symbol.c $(lkc_deps)
#menu.o: menu.c $(lkc_deps)
kconfig_load.o: kconfig_load.c $(lkc_deps)
conf.o: conf.c $(lkc_deps)
mconf.o: mconf.c $(lkc_deps)
gconf.o: gconf.c $(lkc_deps)

mconf: $(mconf_OBJ) lxdialog/lxdialog
	$(HOST_CC) $(HOST_LDFLAGS) $(mconf_OBJ) -o $@ -lncurses

lxdialog/lxdialog:
	$(MAKE) -C lxdialog

conf: $(conf_OBJ)
	$(HOST_CC) $(HOST_LDFLAGS) $^ -o $@

gconf.o: gconf.c
	$(HOST_CC) $(HOST_CFLAGS) $(GTKCFLAGS) -c $< -o $@

gconf: $(gconf_OBJ)
	$(HOST_CC) $(HOST_LDFLAGS) $(GTKLDFLAGS) $^ -o $@

libkconfig.so: $(parse_OBJ)
	$(HOST_CC) -shared $^ -o $@

lkc_defs.h: lkc_proto.h
	sed < $< > $@ 's/P(\([^,]*\),.*/#define \1 (\*\1_p)/'

clean:
	rm -f $(OBJ) conf mconf lex.* *.tab.? *.output qbuild
	rm -rf .ruby .python lkc_defs.h
	$(MAKE) -C lxdialog clean

veryclean: clean
	rm -rf *~ TAGS
tgz:
	mkdir tmp
	mkdir tmp/lkc-$(VERSION)
	cp -ra Makefile $(sort $(SRC) $(HDR) $(OTHER) $(INST)) tmp/lkc-$(VERSION)
	tar -cpvz -C tmp --exclude CVS -f lkc-$(VERSION).tar.gz lkc-$(VERSION)
	rm -rf tmp

%.tab.c %.tab.h: %.y
	$(YACC) $(YFLAGS) -b $* -p $* $<

lex.%.c: %.l
	$(LEX) $(LFLAGS) -P$* $<

%.o: %.c
	$(HOST_CC) $(HOST_CFLAGS) -c $< -o $@

%.o: %.cc
	$(HOST_CXX) $(HOST_CXXFLAGS) -c $< -o $@

ifdef KERNELSRC
install: $(INSTGEN)
	set -x; cp --remove-destination $(sort $(INST)) $(KERNELSRC)/scripts/kconfig; \
	for f in $(INSTGEN); do cp --remove-destination $$f $(KERNELSRC)/scripts/kconfig/$${f}_shipped; done; \
	cp --remove-destination Makefile.kernel $(KERNELSRC)/scripts/kconfig/Makefile; \
	cp --remove-destination lkc-language.txt $(KERNELSRC)/Documentation/kbuild/kconfig-language.txt

diff: $(INSTGEN)
	for f in $(sort $(INST)); do diff -Nu $(KERNELSRC)/scripts/kconfig/$$f $$f; done; \
	for f in $(INSTGEN); do diff -Nu $(KERNELSRC)/scripts/kconfig/$${f}_shipped $$f; done; \
	diff -Nu $(KERNELSRC)/scripts/kconfig/Makefile Makefile.kernel; \
	diff -Nu lkc-language.txt $(KERNELSRC)/Documentation/kbuild/kconfig-language.txt
else
install:
	@echo "Please use KERNELSRC=<path/to/linux-kernel> to install"
endif

ruby: .ruby libkconfig.so .ruby/kconfig.so

.ruby:
	mkdir .ruby

.ruby/kconfig_wrap.c: kconfig.i kconfig_load.c expr.h lkc_proto.h
	swig -ruby -o $@ $<

.ruby/Makefile: extconf.rb
	cd .ruby; ruby ../extconf.rb

.ruby/kconfig.so: .ruby/kconfig_wrap.c .ruby/Makefile
	make -C .ruby


PYTHON_INCLUDE=$(shell python -c "import sys; print '-I'+sys.prefix+'/include/python'+sys.version[:3]")

python: .python .python/kconfig.py .python/_kconfig.so

.python:
	mkdir .python

.python/kconfig_wrap.c .python/kconfig.py: kconfig.i kconfig_load.c expr.h lkc_proto.h
	swig -python -o .python/kconfig_wrap.c kconfig.i

.python/_kconfig.so: .python/kconfig_wrap.c
	cd .python; $(HOST_CC) $(HOST_CFLAGS) -shared kconfig_wrap.c -o _kconfig.so -I.. $(PYTHON_INCLUDE)


.PHONY: all tgz clean ruby python
