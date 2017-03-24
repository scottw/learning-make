find = $(shell find "$(1)" -type f -name "$(2)" -print)

find_pm = $(call find,lib,"*.pm")

test: $(find_pm)
	@echo ok 1
	@echo ok 2
	@touch test

clean:
	@rm test
