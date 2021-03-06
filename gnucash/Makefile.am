if WITH_PYTHON
   PYTHON_DIR = python
endif
# Order is important here.
SUBDIRS = \
  ${PYTHON_DIR} \
  gnome-utils \
  html \
  gnome-search \
  report \
  register \
  gnome \
  import-export \
  .

AM_CPPFLAGS = -I${top_builddir} ${GLIB_CFLAGS} ${GNOME_CFLAGS} ${GTK_CFLAGS} \
  -DPKGSYSCONFDIR=\"${GNC_CONFIGDIR}\" \
  -DPKGDATADIR=\"${GNC_SHAREDIR}\" \
  -I${top_srcdir}/common \
  -I${top_builddir}/common \
  -I${top_srcdir}/libgnucash/core-utils \
  -I${top_srcdir}/libgnucash/app-utils \
  -I${top_srcdir}/gnucash/gnome-utils \
  -I${top_srcdir}/libgnucash/engine \
  -I${top_srcdir}/gnucash/gnome \
  -I${top_builddir}/common \
  -I${top_builddir}/libgnucash/core-utils \
  -I${top_srcdir}/libgnucash/gnc-module \
  -I${top_srcdir}/gnucash/report/report-system \
  ${GUILE_CFLAGS} \
  ${GTK_MAC_CFLAGS}

SUFFIXES = .rc

config_DATA = environment
configdir = ${GNC_CONFIGDIR}

# Some settings are platform dependent. Let's define them per platform.
if PLATFORM_WIN32
# Windows specific settings go here:
GNUCASH_RESOURCE_FILE = gnucash.rc
dist_noinst_DATA = gnucash.rc

.rc.o:
	$(AM_V_GEN)$(RC) -I${top_srcdir}/data/pixmaps -i '$<' --input-format=rc -o '$@' -O coff

else !PLATFORM_WIN32
# All other platforms use these settings:
PLATFORM_FILES = gnucash-valgrind

endif !PLATFORM_WIN32

# gresource for css
gresource_files = $(shell $(GLIB_COMPILE_RESOURCES) --sourcedir=$(srcdir) --generate-dependencies $(srcdir)/gnucash-gresources.xml)
gnucash-gresources.c: gnucash-gresources.xml $(gresource_files)
	$(AM_V_GEN) $(GLIB_COMPILE_RESOURCES) --target=$@ --sourcedir=$(srcdir) --generate-source $<

BIN_NAME = gnucash
bin_PROGRAMS = ${BIN_NAME}
gnucash_SOURCES = gnucash-bin.c gnucash-gresources.c ${GNUCASH_RESOURCE_FILE}
gnucash_LDADD = \
  ${top_builddir}/gnucash/register/ledger-core/libgncmod-ledger-core.la \
  ${top_builddir}/gnucash/report/report-gnome/libgncmod-report-gnome.la \
  ${top_builddir}/gnucash/gnome/libgnc-gnome.la \
  ${top_builddir}/gnucash/gnome-utils/libgncmod-gnome-utils.la \
  ${top_builddir}/libgnucash/app-utils/libgncmod-app-utils.la \
  ${top_builddir}/libgnucash/engine/libgncmod-engine.la \
  ${top_builddir}/libgnucash/gnc-module/libgnc-module.la \
  ${top_builddir}/libgnucash/core-utils/libgnc-core-utils.la \
  ${top_builddir}/gnucash/report/report-system/libgncmod-report-system.la \
  ${GUILE_LIBS} \
  ${GLIB_LIBS} \
  ${GTK_LIBS}

if WITH_GOOGLE_PROFILER
gnucash_LDADD += -lprofiler
endif

GNUCASH_BIN_INSTALL_NAME=`echo ${BIN_NAME} | sed -e '$(transform)'`

gnucash-valgrind: gnucash-valgrind.in ${top_builddir}/config.status Makefile
	rm -f $@.tmp
	sed < $< > $@.tmp \
	    -e "s#[@]TOP_SRC_DIR[@]#${abs_top_srcdir}#g" \
	    -e "s#[@]GNUCASH_BIN_INSTALL_NAME[@]#${GNUCASH_BIN_INSTALL_NAME}#g"
	mv $@.tmp $@
	chmod u+x $@

environment: environment.in ${top_builddir}/config.status Makefile
	rm -f $@.tmp
	sed < $< > $@.tmp \
	    -e '/[@]NOTE.*[@]/ D' \
	    -e "s#[@]GUILE_EFFECTIVE_VERSION[@]#@GUILE_EFFECTIVE_VERSION@#g"
if CUSTOM_GNC_DBD_DIR
	echo 'GNC_DBD_DIR=@GNC_DBD_DIR@' >> $@.tmp
endif
    # Set XDG_DATA_DIRS if necessary.  The three components of the search path are the
    # directory used by GnuCash, whatever was specified in the environment at run time, and
    # the default value specified via configure.
	if [ "a$(datadir)" != "a/usr/share" ] && [ "a$(datadir)" != "a/usr/local/share" ]; \
	then \
		echo >> $@.tmp; \
		echo "# GnuCash was not installed in the default location" >> $@.tmp; \
		echo "# XDG_DATA_DIRS will be set so that our documentation" >> $@.tmp; \
		echo "# and gsettings schemas are found." >> $@.tmp; \
		echo "XDG_DATA_DIRS=$(datadir);{XDG_DATA_DIRS};${GNC_SYSTEM_XDG_DATA_DIRS}" >> $@.tmp; \
	fi
if WITH_PYTHON
	if [ "${PYTHON_SITE_PKG}" != "${pyexecdir}" ]; \
	then \
		echo  >> $@.tmp; \
		echo "# Define PYTHONPATH for non default installation path." >> $@.tmp; \
		echo "PYTHONPATH=${pyexecdir};{PYTHONPATH}" >> $@.tmp; \
	fi
endif
	mv $@.tmp $@

CLEANFILES = $(BUILT_SOURCES) ${config_DATA} ${PLATFORM_FILES}


EXTRA_DIST = \
	generate-gnc-script \
	gnucash-valgrind.in \
	environment.in \
	gnucash-gresources.xml \
	${gresource_files} \
	CMakeLists.txt

AM_CPPFLAGS += -DG_LOG_DOMAIN=\"gnc.bin\"
