## Process this file with automake to produce Makefile.in	-*- makefile -*-
#   $Source: bitbucket.org:berkeleylab/gasnet.git/Makefile.am $
# Description: Makefile for GASNet 
# Copyright 2002, Dan Bonachea <bonachea@cs.berkeley.edu>
# Terms of use are as specified in license.txt

AUTOMAKE_OPTIONS = foreign 1.4
ACLOCAL_AMFLAGS = -I other/plpa/config

# Directories for automake to recurse into:
SUBDIRS =            \
	config-aux   \
        other

# GASNET_TOOLSONLY_TRUE_BEGIN
# CONDUITMODE_DIST_SUBDIRS = 
# CONDUITMODE_headers = 
# CONDUITMODE_EXTRA_DIST = 	\
# 	.gasnet_toolsonly_mode 
# GASNET_TOOLSONLY_TRUE_END
# GASNET_TOOLSONLY_FALSE_BEGIN
  # Directories we manually recurse into:
CONDUITMODE_DIST_SUBDIRS =    	\
	mpi-conduit  		\
	shmem-conduit  		\
	smp-conduit  		\
	udp-conduit  		\
	ibv-conduit 		\
	mxm-conduit		\
	psm-conduit		\
	portals4-conduit        \
	gemini-conduit          \
	aries-conduit           \
	pami-conduit			\
	ofi-conduit
CONDUITMODE_headers = 		\
	gasnet.h           	\
	gasnet_ammacros.h  	\
	gasnet_coll.h      	\
	gasnet_handler.h   	\
	gasnet_trace.h     	\
	gasnet_vis.h      
CONDUITMODE_EXTRA_DIST =    	\
	docs              	\
	extended-ref      	\
	template-conduit  	\
	gasnet_internal.h 	\
	gasnet_internal.c 	\
	gasnet_mmap.c  	  	\
	gasnet_diagnostic.c 	\
	gasnet_pshm.h		\
	gasnet_pshm.c		\
	gasnet_syncops.h   	\
	gasnet_trace.c
# GASNET_TOOLSONLY_FALSE_END

# bug3285: Order matters here: other should follow conduits to maintain dependencies for distclean
DIST_SUBDIRS = $(CONDUITMODE_DIST_SUBDIRS) $(SUBDIRS)

configdir = $(includedir)
config_DATA = $(CONFIG_HEADER)
if NEED_DOCDIR
docdir = $(datadir)/doc/GASNet
endif

headers =                  	\
        $(CONDUITMODE_headers) 	\
	gasnet_asm.h       	\
	gasnet_atomicops.h 	\
	gasnet_atomic_bits.h 	\
        gasnet_basic.h     	\
        gasnet_help.h      	\
	gasnet_membar.h    	\
        gasnet_timer.h     	\
	gasnet_tools.h     	\
	gasnet_toolhelp.h  	

EXTRA_DIST =              		\
	README				\
	README-devel			\
	README-git			\
	README-release			\
 	README-tools 			\
	ChangeLog	  		\
	license.txt       		\
	.autom4te.cfg     		\
        Bootstrap         		\
        unBootstrap       		\
        configure.in      		\
        acconfig.h        		\
        acinclude.m4      		\
        $(headers)        		\
        gasnet_tools.c    		\
	$(CONDUITMODE_EXTRA_DIST)
#       tests handled below

include_HEADERS = $(headers)

INSTALL_DOCS =			\
	README			\
	README-devel		\
	README-git		\
	README-release		\
	README-tools		\
	license.txt		\
	docs/autotuner.txt	\
	docs/pshm-design.txt	\
	docs/gasnet.txt		\
	docs/gasnet.html	\
	docs/gasnet.pdf		\
	docs/gasnet.ps	

force:

#-------------------------------------------------------
# libgasnet_tools goop

if BUILD_SEQ_LIBS
libgasnet_tools_seq_a_SOURCES =
libraries_seq = libgasnet_tools-seq.a 
fragments_seq = gasnet_tools-seq.mak
endif
if BUILD_PAR_LIBS
libgasnet_tools_par_a_SOURCES =
libraries_par = libgasnet_tools-par.a 
fragments_par = gasnet_tools-par.mak
endif
libraries = $(libraries_seq) $(libraries_par)
fragments = $(fragments_seq) $(fragments_par)

# leading - prevents distclean failures
-include $(top_builddir)/other/Makefile-libgasnet.mak

libgasnet_tools-seq.a: $(libgasnet_tools_dependencies)
	@$(MAKE) do-libgasnet_tools-seq

libgasnet_tools-par.a: $(libgasnet_tools_dependencies)
	@$(MAKE) do-libgasnet_tools-par

lib_LIBRARIES = $(libraries)
#-------------------------------------------------------

# bug1758: cannot depend on $(top_builddir)/other/Makefile-libgasnet.mak here,
#  as that will cause a failure during distclean on some make versions
#  make should automatically introduce the desired dependency on Makefile-libgasnet.mak.in
#  due to the include operation above
Makefile: $(srcdir)/Makefile.in 
	cd $(top_builddir) && CONFIG_FILES=$@ CONFIG_HEADERS= ./config.status

# echo an arbitrary make/environment variable
echovar: force
	@echo $(VARNAME)=\"$($(VARNAME))\"

if CONDUIT_MODE
test: tests-seq
tests: tests-seq

other-tests:
	@cd $(top_builddir)/other && $(MAKE) all tests

tests-seq tests-parsync tests-par: other-tests
	@for conduit in $(CONDUITS) ; do                                       \
	  ( cd "$(top_builddir)/$$conduit-conduit" && $(MAKE) $@ ) || exit $$?; \
	done

TESTINSTANCE=$${PPID:-xxxx}
TESTLOG=`pwd`/.test-results


run-tests run-tests-seq run-tests-parsync run-tests-par: do-begin-tests
	@testinst=$(TESTINSTANCE) ; 							\
	testlog=$(TESTLOG) ; 								\
        err=0 ; 									\
	for conduit in $(CONDUITS) ; do                                        		\
	  ( cd "$(top_builddir)/$$conduit-conduit" &&                          		\
            $(MAKE) TESTINSTANCE=$$testinst TESTLOG=$$testlog 				\
		    TESTLOG_INHERIT=1 $@ ) || err=$$? ;					\
          if test "$$err" != "0" ; then                             			\
		echo " *-* COMPILE FAILURE: $$conduit-conduit $@ *-*" >> $(TESTLOG) ;   \
		break ; 								\
	  fi ;                        							\
	done ; 										\
        if test "$$err" = "0" ; then  							\
	  ( cd $(top_builddir)/other &&  						\
            $(MAKE) TESTINSTANCE=$$testinst TESTLOG=$$testlog 				\
		    TESTLOG_INHERIT=1 run-tests ) || err=$$? ; 				\
        fi ; 										\
	$(MAKE) TESTINSTANCE=$(TESTINSTANCE) TESTLOG=$(TESTLOG) do-end-tests ; 		\
	exit $$err

do-begin-tests:
	@testinst=$(TESTINSTANCE) ; \
	testlog=$(TESTLOG) ; \
	if test "`sed 1q $$testlog 2> /dev/null`" != "$$testinst" ; then \
	  echo "$$testinst" > $$testlog ; \
	  echo Starting a new test instance, log=$$testlog ; \
	fi 

do-end-tests:
	@testinst=$(TESTINSTANCE) ; \
	testlog=$(TESTLOG) ; \
	if test "`sed 1q $$testlog 2> /dev/null`" = "$$testinst" ; then \
	  line='+++++++++++++++++++++++++++++++++++++++' ; line="$$line$$line" ; \
	  echo "$$line" ; \
	  echo Test results: `sed -n '1d;/\*-\*/p' $$testlog | wc -l` failures ; \
	  sed 1d $$testlog ; \
	  echo "$$line" ; \
	fi
CLEANFILES = $(TESTLOG) 
else
TOOLSONLY_CHECK = run-tests
TOOLSONLY_TESTS = testtools
test: tests
tests: force
	@$(MAKE) -f $(top_builddir)/tests/Makefile \
	        GASNET_INCLUDES="-I@TOP_BUILDDIR@ -I@TOP_SRCDIR@ -I@TOP_SRCDIR@/other" \
		$(TOOLSONLY_TESTS)
run-tests: tests
	./testtools
CLEANFILES = $(TOOLSONLY_TESTS)
endif

# Build, clean, install, uninstall, etc. the conduits - in parallel if possible.
# Instead of automake's loop over subdir, we expand to one target per subdir.
# Care is taken w/ the 'clean-local' rule in case distclean has wiped the 
# local or conduit Makefiles
# Note that if ($VAR:%=%FOO) doesn't work for a given make, then we are
# reduced to using actual conduit names as PHONY targets, but the rule for
# $(recurse_local) is tailored to work correctly regardless.
recurse_local = $(CONDUITS:%=%-recurse) recurse_local_dummy
$(recurse_local):
	@if test "$@" != "recurse_local_dummy" ; then \
	  cd `echo $@ | sed s/-recurse//`-conduit && $(MAKE) $(DO_WHAT) || exit $$? ; \
	 fi
# conduit subdirectories that should be made, even when the conduit is disabled
SPECIAL_SUBDIRS = mpi-conduit/contrib
recurse_special:
	@for dir in $(SPECIAL_SUBDIRS) ; do \
	  if test -f $$dir/Makefile ; then \
	    cd $$dir && $(MAKE) $(DO_WHAT) || exit $$? ; \
	  fi ; \
	 done
all-local: $(lib_LIBRARIES) $(fragments)
	@$(MAKE) $(recurse_local) DO_WHAT=$${DO_WHAT=all}
	@$(MAKE) recurse_special  DO_WHAT=$${DO_WHAT=all}
clean-local:
	rm -f $(fragments)
	@if test -f Makefile -a -f smp-conduit/Makefile; then \
	  $(MAKE) $(recurse_local) DO_WHAT=clean || exit $$? ; \
	 fi
	@if test -f Makefile; then                   \
	  $(MAKE) recurse_special DO_WHAT=clean ;    \
	 fi
check-local: check-exports $(TOOLSONLY_CHECK)
seq par parsync:
	@$(MAKE) $(recurse_local) DO_WHAT=$@
check tests-clean:
	@$(MAKE) $(recurse_local) DO_WHAT=$@
	@cd "$(top_builddir)/other" && $(MAKE) $@

install-data-local: 
	@if test -n "$(DESTDIR)" -a `expr "X$(DESTDIR)" : X/` = 0; then \
          echo "ERROR: \$$DESTDIR must begin with a '/' character."; \
	  exit 1; \
	 fi
	@if test `expr "X$(prefix)" : X/` = 0; then \
          echo "ERROR: \$$prefix must begin with a '/' character."; \
	  exit 1; \
	 fi
	@$(MAKE) $(recurse_local) DO_WHAT=install-data
	@$(MAKE) recurse_special DO_WHAT=install-data
	$(mkinstalldirs) $(DESTDIR)$(docdir)
	if test -f $(top_srcdir)/extended-ref/README ; then \
	  $(INSTALL_DATA) $(top_srcdir)/extended-ref/README $(DESTDIR)$(docdir)/README-extref || exit $$? ; \
	fi
	for file in $(INSTALL_DOCS) ; do \
	  if test -f $(top_srcdir)/$$file ; then \
	    $(INSTALL_DATA) $(top_srcdir)/$$file $(DESTDIR)$(docdir)/`basename $$file` || exit $$? ; \
	  fi ; \
	done
	@list='$(fragments)'; for p in $$list; do \
          if test -f $$p; then \
            filename=`basename $$p`; \
            echo @GASNET_INSTALL_CMD@ " < $$p > $(DESTDIR)$(includedir)/$$filename"; \
            @GASNET_INSTALL_CMD@ < $$p > $(DESTDIR)$(includedir)/$$filename || exit $$? ; \
          else :; fi; \
        done
	for file in @PATCHED_HEADER@ DUMMY ; do \
	  if test -f $$file ; then \
	    $(mkinstalldirs) $(DESTDIR)$(includedir)/patched-headers ; \
	    $(INSTALL_DATA) patched-headers/$$file $(DESTDIR)$(includedir)/patched-headers/$$file || exit $$? ; \
	  fi ; \
	done
install-exec-local:
	@if test -n "$(DESTDIR)" -a `expr "X$(DESTDIR)" : X/` = 0; then \
          echo "ERROR: \$$DESTDIR must begin with a '/' character."; \
	  exit 1; \
	 fi
	@if test `expr "X$(prefix)" : X/` = 0; then \
          echo "ERROR: \$$prefix must begin with a '/' character."; \
	  exit 1; \
	 fi
	@$(MAKE) $(recurse_local) DO_WHAT=install-exec
	@$(MAKE) recurse_special DO_WHAT=install-exec
uninstall-local: 
	@if test -n "$(DESTDIR)" -a `expr "X$(DESTDIR)" : X/` = 0; then \
          echo "ERROR: \$$DESTDIR must begin with a '/' character."; \
	  exit 1; \
	 fi
	@if test `expr "X$(prefix)" : X/` = 0; then \
          echo "ERROR: \$$prefix must begin with a '/' character."; \
	  exit 1; \
	 fi
	@$(MAKE) $(recurse_local) DO_WHAT=uninstall
	@$(MAKE) recurse_special DO_WHAT=uninstall
	rm -Rf $(DESTDIR)$(docdir)
	for file in $(fragments) ; do \
	  rm -f $(DESTDIR)$(includedir)/$$file ; \
	done
	for file in @PATCHED_HEADER@ ; do \
	  rm -f $(DESTDIR)$(includedir)/patched-headers/$$file ; \
	done

# Bug3285: the behavior of distclean-recursive varies in incompatible ways
# across automake versions, such that no matter what variables we specify
# some versions will break the dependencies between subdirectories.
# Very early automakes ignore DIST_SUBDIRS entirely.
# We override it with our own top-level rule to ensure consistent ordering.
distclean: force
	@for dir in $(DIST_SUBDIRS); do                     \
	   if test -f "$$dir/Makefile"; then                \
	     ( cd "$$dir" && $(MAKE) distclean ) || exit 1; \
	   fi;                                              \
	 done
	$(MAKE) recurse_special DO_WHAT=distclean
	$(MAKE) $(AM_MAKEFLAGS) distclean-am
	rm -f tests/Makefile
	rm -f $(am__CONFIG_DISTCLEAN_FILES)
	rm -f config.status gasnet_config.h stamp-h1
	rm -f Makefile

.PHONY: force tests test do-begin-tests do-end-tests                  \
        other-tests other-run-tests tests-seq tests-par tests-parsync \
        run-tests-seq run-tests-par run-tests-parsync                 \
	recurse_special $(recurse_local)

if TOOLSONLY_MODE
TESTDIST = README testtools.c test.h Makefile.in
else
TESTDIST = `find . -print`
endif

dist-hook:
	if test -d $(srcdir)/.git ; then \
	  GASNET_DESC=`$${GIT=git} --git-dir=$(srcdir)/.git describe` || exit $$? ; \
	  $(PERL) -pi -e "s/no-version-control-info/$$GASNET_DESC/;" -- $(distdir)/configure || exit $$? ; \
	  echo $$GASNET_DESC > $(distdir)/version.git || exit $$? ; \
	fi
	if test -f $(distdir)/docs/Makefile ; then \
          cd $(distdir)/docs && $(MAKE) dist-hook || exit $$? ; \
        fi
	mkdir -p $(distdir)/tests  # need this special hack to handle tests or distcheck will fail
	fulldistdir=`cd $(distdir) ; @PWD_PROG@` export fulldistdir ; \
        cd $(srcdir)/tests ; for file in $(TESTDIST) ; do \
 	  if test -d $$file -a ! -d $$fulldistdir/tests/$$file ; then \
	     mkdir -p $$fulldistdir/tests/$$file || exit $$? ; \
          fi; \
	  if test -f $$file -a ! -f $$fulldistdir/tests/$$file ; then \
	     cp -p $$file $$fulldistdir/tests/$$file || exit $$? ; \
          fi ; \
        done

# If we've enabled PSHM, also enabled for 'make distcheck'
if USE_PSHM
PSHM_CONFIG_FLAG=--enable-pshm
else
PSHM_CONFIG_FLAG=
endif
DISTCHECK_CONFIGURE_FLAGS = $(PSHM_CONFIG_FLAG)

