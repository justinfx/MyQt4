
ROOT=$(CURDIR)
TMPDIR=$(CURDIR)/BUILD
INSTALLDIR=$(TMPDIR)/install
PACKAGEDIR=$(TMPDIR)/package

MAYA_LOCATION=/Applications/Autodesk/maya2012/Maya.app/Contents
MAYAPY=$(MAYA_LOCATION)/bin/mayapy

XDATE=`date +%Y%m%d`
JOBS=8


SRC_PYQT_NAME=PyQt-mac-gpl-4.8.6
SRC_PYQT=http://distfiles.macports.org/python/PyQt-mac-gpl-4.8.6.tar.gz

SRC_SIP_NAME=sip-4.12.4
SRC_SIP=http://tx-us.lunar-linux.org/lunar/mirrors/sip-4.12.4.tar.gz

SRC_QT_NAME=qt-adsk-4.7.1
SRC_QT=http://images.autodesk.com/adsk/files/Qt-4.7.1-Modified_for_Maya.zip

# to make sure PyQt4 picks up our sip install
export PYTHONPATH:= $(INSTALLDIR)/Frameworks/Python.framework/Versions/Current/lib/python2.6/site-packages:$(PATH)

export DYLD_LIBRARY_PATH:= $(INSTALLDIR)/lib



build: prepare qt sip pyqt

all: fetch BUILD_PACKAGE



prepare:
	-rm -rf $(INSTALLDIR)
	mkdir -p $(INSTALLDIR)


fetch: prepare
	-mkdir $(TMPDIR)
	cd $(TMPDIR) && \
	if [ -e "qt-adsk-4.7.1" ]; then \
		echo " *** qt-adsk-4.7.1 is already present"; \
	else \
		echo " *** Downloading Qt Modified for Maya"; \
		curl -O $(SRC_QT); \
		unzip Qt-4.7.1-Modified_for_Maya.zip; \
		mv Qt-4.7.1-Maya/qt-adsk-4.7.1.tgz . ; \
		tar zxf qt-adsk-4.7.1.tgz; \
	fi 
	
	cd $(TMPDIR) && \
	for component in $(SRC_SIP) $(SRC_PYQT); do \
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
	echo "\n *** Working on Qt"
	cd $(TMPDIR)/$(SRC_QT_NAME) && \
	yes | ./configure -opensource -silent -arch x86_64 -debug-and-release -no-phonon \
		-no-phonon-backend -no-qt3support -no-webkit -nomake docs -no-libmng -no-accessibility \
		-nomake examples -nomake demos -nomake translations -no-rpath -no-framework -no-sql-sqlite \
		-no-declarative -no-javascript-jit -no-scripttools -no-script -no-declarative-debug \
		-prefix $(INSTALLDIR) && \
	\
	cd $(TMPDIR)/$(SRC_QT_NAME) && make -j$(JOBS) && make install;


sip: fetch
	echo "\n *** Working on sip"
	cd $(TMPDIR)/$(SRC_SIP_NAME) && \
	$(MAYAPY) configure.py --arch=x86_64 \
	   -b $(INSTALLDIR)/Frameworks/Python.framework/Versions/Current/bin \
	   -d $(INSTALLDIR)/Frameworks/Python.framework/Versions/Current/lib/python2.6/site-packages \
	   -e $(INSTALLDIR)/Frameworks/Python.framework/Versions/Current/include/python2.6 \
	   -v $(INSTALLDIR)/Frameworks/Python.framework/Versions/Current/share/sip && \
	\
	cd $(TMPDIR)/$(SRC_SIP_NAME) && make -j$(JOBS) && make install;


pyqt: qt sip
	echo "\n *** Working on PyQt4"
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
	$(pyqt-fix)


pyqt-fix:
	for mod in Core Gui Svg OpenGL Xml; do \
		find $(INSTALLDIR)/Frameworks/Python.framework/Versions/Current/lib/python2.6/site-packages/PyQt4 \
		-name "*$${mod}.so" -exec install_name_tool \
		-change libQt$${mod}.4.dylib @executable_path/Qt$${mod} {} \;; \
	done && \
	\
	sed 's/pythonw2\.6/$$(MAYA_LOCATION)\/bin\/mayapy/' $(INSTALLDIR)/Frameworks/Python.framework/Versions/Current/bin/pyuic4 \
		> $(INSTALLDIR)/Frameworks/Python.framework/Versions/Current/bin/myuic4 && \
	chmod 777 $(INSTALLDIR)/Frameworks/Python.framework/Versions/Current/bin/myuic4
	-mkdir -p $(INSTALLDIR)/usr/local/bin
	cp $(INSTALLDIR)/Frameworks/Python.framework/Versions/Current/bin/myuic4 $(INSTALLDIR)/usr/local/bin


BUILD_PACKAGE: build
	$(package)
	
package: 
	echo " *** Creating package"
	if [ -e $(PACKAGEDIR) ]; then \
		sudo rm -rf $(PACKAGEDIR); \
	fi
	mkdir -p $(PACKAGEDIR)/usr
	mkdir -p $(PACKAGEDIR)/$(MAYA_LOCATION)/Frameworks
	\
	PKG_FILE="MyQt4.8.6-maya2012-x64-osx.pkg"; \
	echo " *** Staging $$PKG_FILE" ; \
	cp -a $(INSTALLDIR)/usr/* $(PACKAGEDIR)/usr/ && \
	cp -a $(INSTALLDIR)/Frameworks/* $(PACKAGEDIR)/Frameworks/ && \
	\
	cd $(TMPDIR) && \
	echo " *** Creating pkg" && \
	rm -f $$PKG_FILE && \
	sudo chown -R root:admin $(PACKAGEDIR) && \
	/Developer/usr/bin/packagemaker --root $(PACKAGEDIR) --out $$PKG_FILE --version "4.8.6" --title MyQt4 --install-to / --domain system --target 10.5 --filter "\.DS_Store" --root-volume-only --id com.justinfx.myqt4 && \
	\
	xar -xf $$PKG_FILE Bom && \
	lsbom package.pkg/Bom && \
	rm -rf package.pkg && \
	mv $$PKG_FILE .. && \
	\
	echo " *** Package $$PKG_FILE built successfully!"; \
	echo " *** Install with command: sudo installer -pkg $$PKG_FILE -target \"/\""


.PHONY: prepare fetch build package qt sip pyqt
