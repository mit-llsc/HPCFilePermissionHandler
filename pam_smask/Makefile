CC = /usr/bin/gcc
RM = /bin/rm
STRIP = /usr/bin/strip
PREFIX = 

ifeq ($(PREFIX),)
    PREFIX := /usr
endif

CFLAGS = -O3 -Wall -std=c99
LFLAGS = 

TARGETBINS = smask_relax
TARGETLIBS = pam_smask.so

all: smask_relax pam_smask.so

pam_smask.so: pam_smask.c
	$(CC) $(CFLAGS) -fPIC -shared -Xlinker -x $< -o $@ $(LFLAGS)
	$(STRIP) --strip-debug --strip-unneeded $@

smask_relax: smask_relax.c
	$(CC) $(CFLAGS) $< -o $@ $(LFLAGS)
	$(STRIP) --strip-debug --strip-unneeded $@

.PHONY: install
install: all
	install -m 0755 -d $(DESTDIR)$(PREFIX)/bin
	install -m 0755 $(TARGETBINS) $(DESTDIR)$(PREFIX)/bin
	install -m 0755 -d $(DESTDIR)$(PREFIX)/lib/x86_64-linux-gnu/security
	install -m 0755 $(TARGETLIBS) $(DESTDIR)$(PREFIX)/lib/x86_64-linux-gnu/security

clean:
	$(RM) -rf pam_smask.so smask_relax debian/pam_smask* debian/.debhelper

deb:
	dpkg-buildpackage -b -rfakeroot -us -uc
