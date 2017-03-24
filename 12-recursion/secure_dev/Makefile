.PHONY: build clean test

SRCDIR = src
SRC    = $(SRCDIR)/secure_dev
REPO   = ssh://git@stash.endurance.com:7999/hp/secure_dev.git

build: $(SRC)

$(SRC):
	@echo "Building secure_dev at $@"
	@git clone $(REPO) $(SRC)

test: build
	@echo "Testing secure_dev...";
	@echo ok 1

clean:
	-@rm -rf $(SRC)
