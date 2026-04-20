all: build/henloader.iso

ARCH := $(shell uname -m)
ifeq ($(ARCH),aarch64)
	JDK8_PACKAGE  := thirdparty/OpenJDK8U-jdk_aarch64_linux_hotspot_8u462b08.tar.gz
else ifeq ($(ARCH),x86_64)
	JDK8_PACKAGE  := thirdparty/OpenJDK8U-jdk_x64_linux_hotspot_8u462b08.tar.gz
else
	$(error Unknown ARCH "$(ARCH)")
endif

JDK8  := build/jdk8
JAVA8 := $(JDK8)/bin/java

$(JAVA8):
	mkdir -p $(JDK8)
	tar -xf $(JDK8_PACKAGE) -C $(JDK8) --strip-components=1

# أدوات التوقيع
BDTOOLS  := thirdparty/bd-tools
SECCPATH := $(BDTOOLS)/security.jar:$(BDTOOLS)/bcprov-jdk15-137.jar:$(BDTOOLS)/jdktools.jar
KEYSTORE := thirdparty/bd-certificates/keystore.store
BDSIGNER := $(JAVA8) -cp $(SECCPATH) net.java.bd.tools.security.BDSigner -keystore $(KEYSTORE)

CPATH  := thirdparty/bd-stubs/interactive.zip:thirdparty/topsecret/rt.jar:thirdparty/topsecret/bdjstack.jar:src
JFLAGS := -Xlint:all -Xlint:-static -Xlint:-serial -Xlint:-options -source 1.3 -target 1.3

LOADER_DSTDIR  := build/henloader
LOADER_BD_PERM := org/bdj/bluray.InitXlet.perm

LOADER_SOURCES += org/bdj/sandbox/DisableSecurityManagerAction.java
LOADER_SOURCES += org/bdj/external/BinLoader.java
LOADER_SOURCES += org/bdj/external/Helper.java
LOADER_SOURCES += org/bdj/external/Kernel.java
LOADER_SOURCES += org/bdj/external/KernelOffset.java
LOADER_SOURCES += org/bdj/external/Poops.java
LOADER_SOURCES += org/bdj/external/Lapse.java
LOADER_SOURCES += org/bdj/InitXlet.java
LOADER_SOURCES += org/bdj/MessagesOutputStream.java
LOADER_SOURCES += org/bdj/Screen.java

build/henloader.jar: $(JAVA8) $(addprefix src/,$(LOADER_SOURCES)) src/$(LOADER_BD_PERM)
	mkdir -p $(LOADER_DSTDIR)
	mkdir -p $(LOADER_DSTDIR)/$(dir $(LOADER_BD_PERM))
	cp src/$(LOADER_BD_PERM) $(LOADER_DSTDIR)/$(LOADER_BD_PERM)
	mkdir -p $(LOADER_DSTDIR)/org/homebrew/
	$(JDK8)/bin/javac -d $(LOADER_DSTDIR) -sourcepath src $(JFLAGS) -cp $(CPATH) $(addprefix src/,$(LOADER_SOURCES))
	$(JDK8)/bin/jar cf $@ -C $(LOADER_DSTDIR) .
	$(BDSIGNER) $@
	-rm META-INF/SIG-BD00.RSA
	-rm META-INF/SIG-BD00.SF
	-rmdir META-INF

# Blu-ray structure
DISC      := build/disc
BD_JO     := $(DISC)/BDMV/BDJO/00000.bdjo
BD_JAR    := $(DISC)/BDMV/JAR/00000.jar
BD_FONT   := $(DISC)/BDMV/AUXDATA/00000.otf
BD_FNTIDX := $(DISC)/BDMV/AUXDATA/dvb.fontindex
BD_HEN    := $(DISC)/BDMV/AUXDATA/aiofix_USBpayload.elf
BD_META   := $(DISC)/BDMV/META/DL/bdmt_eng.xml
BD_BANNER := $(DISC)/BDMV/META/DL/logo.jpg
BD_INDEX  := $(DISC)/BDMV/index.bdmv
BD_MVOBJ  := $(DISC)/BDMV/MovieObject.bdmv
BD_ID     := $(DISC)/CERTIFICATE/id.bdmv
BD_ACRT   := $(DISC)/CERTIFICATE/app.discroot.crt
BD_BCRT   := $(DISC)/CERTIFICATE/bu.discroot.crt

BD_ALL := $(BD_JO) $(BD_JAR) $(BD_FONT) $(BD_FNTIDX) $(BD_META) $(BD_BANNER) \
          $(BD_INDEX) $(BD_MVOBJ) $(BD_ID) $(BD_ACRT) $(BD_BCRT) $(BD_HEN)

$(DISC): $(sort $(dir $(BD_ALL)))
$(sort $(dir $(BD_ALL))):
	mkdir -p $(dir $(BD_ALL))

$(BD_JO): bd-metadata/bdjo.xml $(DISC) $(JAVA8)
	$(JAVA8) -jar thirdparty/bd-tools/bdjo.jar $< $@

$(BD_JAR): build/henloader.jar $(DISC)
	cp $< $@

$(BD_FONT): bd-metadata/OpenSans-Regular.otf $(DISC)
	cp $< $@

$(BD_FNTIDX): bd-metadata/dvb.fontindex $(DISC)
	cp $< $@

$(BD_HEN): bd-metadata/aiofix_USBpayload.elf $(DISC)
	cp $< $@

$(BD_META): bd-metadata/bdmt_eng.xml $(DISC)
	cp $< $@

$(BD_BANNER): bd-metadata/logo.jpg $(DISC)
	cp $< $@

$(BD_INDEX): bd-metadata/index.xml $(DISC) $(JAVA8)
	$(JAVA8) -jar thirdparty/bd-tools/index.jar $< $@

$(BD_MVOBJ): bd-metadata/movieobject.xml $(DISC) $(JAVA8)
	$(JAVA8) -jar thirdparty/bd-tools/movieobject.jar $< $@

$(BD_ID): bd-metadata/id.xml $(DISC) $(JAVA8)
	$(JAVA8) -jar thirdparty/bd-tools/id.jar $< $@

$(BD_ACRT): thirdparty/bd-certificates/app.discroot.crt $(DISC)
	cp $< $@

$(BD_BCRT): thirdparty/bd-certificates/bu.discroot.crt $(DISC)
	cp $< $@

# ✅ هنا التعديل المهم (xorriso بدل makefs)

DISC_LABEL := HenLoader

build/henloader.iso: $(BD_ALL)
	mkdir -p build
	xorriso -as mkisofs -iso-level 3 -udf -V $(DISC_LABEL) -o $@ $(DISC)

.PHONY: clean
clean:
	rm -rf build
