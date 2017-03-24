#.PHONY: clean

HTMLDOC = index.html
INPUTS  = index.md style.css doc.js

docs: $(HTMLDOC)

$(HTMLDOC): $(INPUTS)
	md2html -i index.md -o $(HTMLDOC)

clean:
	@rm $(HTMLDOC)
