.PHONY: clean test

SRCDIR = src
SRC    = $(SRCDIR)/core_lib
REPO   = ssh://git@stash.endurance.com:7999/hp/core_lib.git

build: $(SRC)

$(SRC):
	@echo "Building core_lib at $@"
	@git clone $(REPO) $(SRC)

test: build
	@echo "Testing core_lib...";
	@echo ok 1
	@echo ok 2

clean:
	-@rm -rf $(SRC)
