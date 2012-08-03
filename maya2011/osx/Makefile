TMPDIR=$(CURDIR)/BUILD
INSTALLDIR=$(TMPDIR)/install
PACKAGEDIR=$(TMPDIR)/package

MAYA_LOCATION=/Applications/Autodesk/maya2011/Maya.app/Contents
MAYAPY=$(MAYA_LOCATION)/bin/mayapy

XDATE=$(shell date +%Y%m%d)
JOBS=$(shell sysctl hw.physicalcpu | cut -d : -f 2)

SRC_PYQT_NAME=PyQt-mac-gpl-4.7.4
SRC_PYQT=https://dl.dropbox.com/u/34613220/MyQt4/src/PyQt-mac-gpl-4.7.4.tar.gz

SRC_SIP_NAME=sip-4.10.3
SRC_SIP=https://dl.dropbox.com/u/34613220/MyQt4/src/sip-4.10.3.tar.gz

SRC_QT_NAME=qt-mac-opensource-src-4.5.3
SRC_QT=https://dl.dropbox.com/u/34613220/MyQt4/src/qt-mac-opensource-src-4.5.3.tar.gz

PKG_FILE=MyQt4.7.4-maya2011-x64-osx-$(XDATE).pkg

# to make sure PyQt4 picks up our sip install
export PYTHONPATH:= $(INSTALLDIR)/Frameworks/Python.framework/Versions/Current/lib/python2.6/site-packages:$(PATH)

export DYLD_LIBRARY_PATH:= $(INSTALLDIR)/lib


.PHONY: all build build-package
all: build-package

build: prepare qt sip pyqt

build-package: build
	$(MAKE) package -C $(CURDIR)

.PHONY: prepare
prepare:
	-mkdir -p $(INSTALLDIR)

.PHONY: fetch

.PHONY: clean
clean:
	if [ -e "$(INSTALLDIR)" ]; \
		then rm -rf $(INSTALLDIR); \
	fi; 

	if [ -e "$(PACKAGEDIR)" ]; \
		then rm -rf $(PACKAGEDIR); \
	fi; 
	
fetch: prepare
	cd $(TMPDIR) && \
	for component in $(SRC_QT) $(SRC_SIP) $(SRC_PYQT); do \
		f=`echo $$component | awk -F/ '{print $$NF}'`; \
		name=`echo $$f | sed 's/.tar.gz$$//'`; \
		if [ -e $$name ]; then \
			echo " *** $$name is already present"; \
		else \
			echo " *** Downloading $$name"; \
			curl -O $$component; \
			tar zxf $$f; \
		fi \
	done


qt: fetch
	@echo "\n *** Working on Qt"
	cd $(TMPDIR)/$(SRC_QT_NAME) && \
	yes yes | ./configure -opensource -silent -fast -arch x86_64 -cocoa -arch x86_64 \
	-debug-and-release -no-phonon -no-phonon-backend -no-qt3support -no-webkit -nomake docs \
	-nomake examples -nomake demos -nomake translations -no-rpath -no-framework	\
	-prefix $(INSTALLDIR) && \
	\
	cd $(TMPDIR)/$(SRC_QT_NAME) && make -j$(JOBS) && make install;



sip: fetch 
	@echo "\n *** Working on sip"
	cd $(TMPDIR)/$(SRC_SIP_NAME) && \
	$(MAYAPY) configure.py --arch=x86_64 \
	   -b $(INSTALLDIR)/Frameworks/Python.framework/Versions/Current/bin \
	   -d $(INSTALLDIR)/Frameworks/Python.framework/Versions/Current/lib/python2.6/site-packages \
	   -e $(INSTALLDIR)/Frameworks/Python.framework/Versions/Current/include/python2.6 \
	   -v $(INSTALLDIR)/Frameworks/Python.framework/Versions/Current/share/sip && \
	\
	cd $(TMPDIR)/$(SRC_SIP_NAME) && make -j$(JOBS) && make install;


pyqt: qt $(BUILD-SIP)
	@echo "\n *** Working on PyQt4"
	cd $(TMPDIR)/$(SRC_PYQT_NAME) && \
	QTDIR=$$INSTALLDIR \
	QMAKESPEC=$(INSTALLDIR)/mkspecs/macx-g++ \
	DYLD_LIBRARY_PATH=$$DYLD_LIBRARY_PATH \
	PATH=$(INSTALLDIR)/bin:$(PATH) \
	$(MAYAPY) configure.py \
	   LIBDIR_QT=$(INSTALLDIR)/lib \
	   INCDIR_QT=$(INSTALLDIR)/include \
	   MOC=$(INSTALLDIR)/bin/moc \
	   -q $(INSTALLDIR)/bin/qmake \
	   -w --no-designer-plugin --confirm-license \
	   -b $(INSTALLDIR)/Frameworks/Python.framework/Versions/Current/bin \
	   -d $(INSTALLDIR)/Frameworks/Python.framework/Versions/Current/lib/python2.6/site-packages \
	   -v $(INSTALLDIR)/Frameworks/Python.framework/Versions/Current/share/sip && \
	\
	cd $(TMPDIR)/$(SRC_PYQT_NAME) && make -j$(JOBS) && make install && \
	\
	$(MAKE) pyqt-fixes -C $(CURDIR)


.PHONY: pyqt-fixes
pyqt-fixes:
	for mod in Core Gui Svg OpenGL Xml; do \
		find $(INSTALLDIR)/Frameworks/Python.framework/Versions/Current/lib/python2.6/site-packages/PyQt4 \
		-name "*.so" -exec install_name_tool \
		-change libQt$${mod}.4.dylib @executable_path/Qt$${mod} {} \;; \
	done

	@echo "$$MYQT4_TOOL" > $(INSTALLDIR)/Frameworks/Python.framework/Versions/Current/bin/pyuic4 && \
	chmod a+x $(INSTALLDIR)/Frameworks/Python.framework/Versions/Current/bin/pyuic4 && \
	mkdir -p $(INSTALLDIR)/usr/local/bin && \
	cp $(INSTALLDIR)/Frameworks/Python.framework/Versions/Current/bin/pyuic4 $(INSTALLDIR)/usr/local/bin/myuic4

	cp $(INSTALLDIR)/Frameworks/Python.framework/Versions/Current/bin/pyrcc4 $(INSTALLDIR)/usr/local/bin/myrcc4


package:
	@echo " *** Creating package"
	if [ -e $(PACKAGEDIR) ]; then \
		sudo rm -rf $(PACKAGEDIR); \
	fi
	mkdir -p $(PACKAGEDIR)/usr
	mkdir -p $(PACKAGEDIR)/$(MAYA_LOCATION)/Frameworks
	\
	PKG_FILE="$(PKG_FILE)" ;\
	\
	echo " *** Staging $$PKG_FILE" ;\
	cp -a $(INSTALLDIR)/usr/* $(PACKAGEDIR)/usr/ && \
	cp -a $(INSTALLDIR)/Frameworks/* $(PACKAGEDIR)/$(MAYA_LOCATION)/Frameworks/  && \
	cd $(TMPDIR) && \
	\
	echo " *** Creating pkg" && \
	rm -f $$PKG_FILE && \
	sudo chown -R root:admin $(PACKAGEDIR) && \
	/Developer/usr/bin/packagemaker --root $(PACKAGEDIR) --out $$PKG_FILE --id com.justinfx.MyQt4 --version 4.7.4 --title MyQt4 --install-to / --domain system --target 10.5 --filter "\.DS_Store" --root-volume-only && \
	\
	xar -xf $$PKG_FILE Bom && \
	lsbom package.pkg/Bom && \
	rm -rf package.pkg && \
	mv $$PKG_FILE .. && \
	\
	echo " *** Package $$PKG_FILE built successfully!"; \
	echo " *** Install with command: sudo installer -pkg $$PKG_FILE -target \"/\""

define MYQT4_TOOL
#!/bin/sh
MAYA_LOCATION=/Applications/Autodesk/maya2011/Maya.app/Contents
exec ${MAYA_LOCATION}/bin/mayapyw ${MAYA_LOCATION}/Frameworks/Python.framework/Versions/Current/lib/python2.6/site-packages/PyQt4/uic/pyuic.py $${1+"$$@"}
endef

export MYQT4_TOOL

