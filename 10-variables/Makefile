.PHONY: test

PM_FILES = lib/Foo.pm lib/Bar.pm lib/Bar/Baz.pm

test: .last-test

.last-test: $(PM_FILES)
	prove -lv t
	@touch .last-test
