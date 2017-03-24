REPOS = core_lib secure_dev
export SRCDIR := $(CURDIR)/src

.PHONY: test clean

build: $(foreach repo,$(REPOS),build-$(repo))
	@echo TOP LEVEL: all repos have been built!

build-%:
	@echo TOP LEVEL: building $*
	@$(MAKE) -e -C $* build

test: $(foreach repo,$(REPOS),test-$(repo))
	@echo TOP LEVEL: all repos have been tested!

test-%:
	@$(MAKE) -e -C $* test

clean: $(foreach repo,$(REPOS),clean-$(repo))
	@echo TOP LEVEL: all repos have been cleaned!

clean-%:
	@$(MAKE) -e -C $* clean
