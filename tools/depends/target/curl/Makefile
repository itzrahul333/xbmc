include ../../Makefile.include
DEPS= ../../Makefile.include Makefile 0001-HTTP-reset-expected-DL-UL-sizes-on-redirects.patch

# lib name, version
LIBNAME=curl
ifeq (osx, $(findstring osx, $(OS)))
VERSION=7.21.6
else
VERSION=7.56.1
endif
SOURCE=$(LIBNAME)-$(VERSION)
ARCHIVE=$(SOURCE).tar.bz2
# configuration settings
CONFIGURE=cp -f $(CONFIG_SUB) $(CONFIG_GUESS) .; \
          ./configure --prefix=$(PREFIX) \
          --without-libssh2 --disable-ntlm-wb --enable-ipv6 --without-librtmp

LIBDYLIB=$(PLATFORM)/lib/.libs/lib$(LIBNAME).a

CLEAN_FILES=$(ARCHIVE) $(PLATFORM)

all: .installed-$(PLATFORM)

$(TARBALLS_LOCATION)/$(ARCHIVE):
	cd $(TARBALLS_LOCATION); $(RETRIEVE_TOOL) $(RETRIEVE_TOOL_FLAGS) $(BASE_URL)/$(ARCHIVE)

$(PLATFORM): $(TARBALLS_LOCATION)/$(ARCHIVE) $(DEPS)
	rm -rf $(PLATFORM)/*; mkdir -p $(PLATFORM)
	cd $(PLATFORM); $(ARCHIVE_TOOL) $(ARCHIVE_TOOL_FLAGS) $(TARBALLS_LOCATION)/$(ARCHIVE)
ifeq (osx, $(findstring osx, $(OS)))
	cd $(PLATFORM); patch -p1 < ../0001-HTTP-reset-expected-DL-UL-sizes-on-redirects.patch
endif
ifeq (darwin, $(findstring darwin, $(HOST)))
	cd $(PLATFORM); sed -ie "s/elif defined(HAVE_CLOCK_GETTIME_MONOTONIC)/#elif defined(HAVE_CLOCK_GETTIME_MONOTONIC_NOPE)/" lib/timeval.c
endif
	cd $(PLATFORM); $(CONFIGURE)

$(LIBDYLIB): $(PLATFORM)
	$(MAKE) -C $(PLATFORM)/lib

.installed-$(PLATFORM): $(LIBDYLIB)
	# install only libcurl and includes
	$(MAKE) -C $(PLATFORM)/lib install
	$(MAKE) -C $(PLATFORM)/include install
	# make sure to also install libcurl.pc, we need to do
	# this because we are selectively installing only libcurl
	$(MAKE) -C $(PLATFORM) install-pkgconfigDATA
	touch $@

clean:
	$(MAKE) -C $(PLATFORM) clean
	rm -f .installed-$(PLATFORM)

distclean::
	rm -rf $(PLATFORM) .installed-$(PLATFORM)

