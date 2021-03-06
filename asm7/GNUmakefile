BUILDABLEASM := $(sort $(patsubst %.cc,%.s,$(wildcard f*.cc)) \
	$(patsubst src/%.cc,%.s,$(wildcard src/f*.cc)))
ALLASM := $(sort $(BUILDABLEASM) $(wildcard f*.s))
ASMOBJECTS := $(patsubst %.s,%.o,$(ALLASM))
ASMPROGRAMS := $(sort $(patsubst %.s,%,$(patsubst %.cc,%,$(patsubst src/%,%,$(shell grep -l main $(wildcard f*.s f*.cc src/f*.cc /dev/null))))))
DEFAULT_ASM_CXXFLAGS = $(O)
DEFAULT_ASM_CXXFLAGS_DEBUG = -g $(DEFAULT_ASM_CXXFLAGS)
PROGRAMS = stringrep vectorrep $(ASMPROGRAMS)

all: $(BUILDABLEASM) $(PROGRAMS)

asm: cleanasm $(BUILDABLEASM) $(PROGRAMS)

variants = $(1) $(addsuffix -unsafe,$(1)) $(addsuffix -O0,$(1))


COMPILER ?= gcc
PIE ?= 0
O ?= 1
include ../common/rules.mk

UNSAFEFLAGS := -U_FORTIFY_SOURCE -fno-stack-protector $(if $(ISLINUX),-no-pie,)

%.o: %.cc $(BUILDSTAMP)
	$(CXX) $(CPPFLAGS) $(CXXFLAGS) $(O) $(DEPCFLAGS) -o $@ -c $<

%-unsafe.o: %.cc $(BUILDSTAMP)
	$(CXX) $(CPPFLAGS) $(UNSAFEFLAGS) $(CXXFLAGS) -O1 $(DEPCFLAGS) -o $@ -c $<

%-O0.o: %.cc $(BUILDSTAMP)
	$(CXX) $(CPPFLAGS) $(CXXFLAGS) -O0 $(DEPCFLAGS) -o $@ -c $<

%-O1.o: %.cc $(BUILDSTAMP)
	$(CXX) $(CPPFLAGS) $(CXXFLAGS) -O1 $(DEPCFLAGS) -o $@ -c $<


%.s: %.cc GNUmakefile $(BUILDSTAMP)
	$(call flagged_compile_S,$<,$@,$(CXXFLAGS))

%.s: src/%.cc GNUmakefile $(BUILDSTAMP)
	$(call flagged_compile_S,$<,$@,-I. $(CXXFLAGS))

%-unsafe.s: %.cc GNUmakefile $(BUILDSTAMP)
	$(call flagged_compile_S,$<,$@,$(UNSAFEFLAGS) $(CXXFLAGS))

%-unsafe.s: src/%.cc GNUmakefile $(BUILDSTAMP)
	$(call flagged_compile_S,$<,$@,-I. $(UNSAFEFLAGS) $(CXXFLAGS))

%.asm: %.o
	objdump -d $< > $@


%.o: %.cc $(BUILDSTAMP)
	$(CXX) $(CPPFLAGS) $(CXXFLAGS) $(O) $(DEPCFLAGS) -o $@ -c $<

$(ASMOBJECTS): %.o: %.s $(BUILDSTAMP)
	$(call run,$(CXX) -o $@ -c,ASSEMBLE,$<)

$(PROGRAMS): %: %.o hexdump.o
	$(CXX) $(CXXFLAGS) $(LDFLAGS) $(O) -o $@ $^
	@if grep -l '^//@disassemble' src/$@.cc >/dev/null 2>&1; then objdump --disassemble=`grep '^//@disassemble' src/$@.cc | cut -d ' ' -f 2` $@ > $@.s; fi


allfc.txt: always
	for i in $(foreach x,$(patsubst %.s,%.cc,$(BUILDABLEASM)),$(wildcard $(x) src/$(x))); do \
	    echo $$i; echo ========; cat $$i; echo; echo; done > allfc.txt

allfs.txt: $(ALLASM) always
	for i in $(ALLASM); do \
	    echo $$i; echo ========; cat $$i; echo; echo; done > allfs.txt


cleanasm clean:
	rm -rf $(call variants,$(PROGRAMS)) *.o $(DEPSDIR) $(BUILDABLEASM)

.PHONY: all clean cleanasm asm
.PRECIOUS: %-O0.o %-O1.o %-unsafe.o %-unsafe.s
