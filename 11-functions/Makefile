.PHONY: touch clean realclean

PM_FILES = $(shell find lib -name "*.pm" 2>/dev/null)
T_FILES  = $(shell find t   -name "*.t"  2>/dev/null)

test: .last-test

.last-test: $(PM_FILES) $(T_FILES)
	prove -lv t
	@touch .last-test

clean:
	-@rm .last-test

realclean: clean
	-@rm -r lib

## convenience function for the presentation
touch:
	-@mkdir -p lib/Bar
	touch lib/Foo.pm lib/Bar.pm lib/Bar/Baz.pm
