TARGET := paper

#------------------------------------------------------------------------
# This shell script extracts all files given as an argument to \input{},
# starting with the TARGET tex file, and stores this list of tex files in
# TEX_ALL
#------------------------------------------------------------------------
TEX_ALL := $(shell search=$(TARGET).tex; all=; \
				while [ -n "$$search" ] ; do \
					all="$$all $$search"; \
					search=`egrep "^[^%]*\\input" $$search | \
						sed -En 's/.*input[^\{]*\{(.+)\}/\1.tex/p'`; \
				done; \
				echo "$$all")


#------------------------------------------------------------------------
# This shell script greps each input tex file and extracts all
# all file arguments to \includegraphics, and stores this list of graphic
# files in FIGURES.
#
# Note, we assume that these are all .eps files.
#
# The convention is to place gnuplot files in plots/, this Makefile then invokes
# the gnuplot scripts and writes the eps ouput files to figs/.
# To include such an .eps file, say, figs/bar.eps, use:
# 
# 	\includegraphics{figs/bar}
# 
# For PDFs (say, a diagram you
# made in Keynote), place the PDF elsewhere (say diagrams/) and specify the
# .pdf extension (e.g., includegraphics{diagrams/baz.pdf})
#------------------------------------------------------------------------
FIGURES := $(shell for t in $(TEX_ALL); do \
				cat $$t | \
				egrep '^[^%]*\\includegraphics' | \
				sed -En 's/.*includegraphics(\[.+\])?\{([^\{]*)\}.*/\2/p'; \
				done)

# remove the .pdf files that we create (e.g., with KeyNote)
FIGURES := $(addsuffix .eps, $(filter-out %.pdf,$(FIGURES)))

all: $(TARGET).pdf

figs/%.eps: plots/%.gpi
	gnuplot $< > $@

$(TARGET).pdf: $(FIGURES) $(TEX_ALL) conferences.bib refs.bib
	pdflatex $(TARGET).tex
	bibtex $(TARGET)
	pdflatex $(TARGET).tex
	pdflatex $(TARGET).tex
	@nn=`grep "Warning: Citation" $(TARGET).log | sort | uniq | wc -l | awk '{print $$1;}'`; if [[ $$nn -gt 0 ]] ; then echo "\033[1;33m################################\n$$nn missing citations:"; grep "Warning: Citation" $(TARGET).log | awk '{print "\033[0;31m   ", $$4;}' | sed "s/[\`']//g" | sort | uniq; echo "\033[1;33m################################\033[0m"; else echo "\033[0;34mZero missing citations! 👏🎉\033[0m"; fi

nobib:: $(addsuffix .eps, $(FIGURES)) $(TARGET).aux
	dvips $(TARGET).dvi -Ppdf -Pcmz -Pamz -t letter -D 600 -G0 -o $(TARGET).ps
	ps2pdf ${PS2PDF_FLAGS} $(TARGET).ps


%.eps: plots/%.gpi
	gnuplot $< > figs/$@

%.eps: figs/%.fig
	fig2dev -L eps -p dummy $< $*.eps

clean::
	rm -fv *.dvi *.aux *.log *~ *.bbl *.blg *.toc *.out *.ps *.pdf *.ent parsetab.py
	rm -fv $(addsuffix -eps-converted-to.pdf, $(FIGURES))

fresh::
	rm -fv *.dvi *.aux *.log *~ *.bbl *.blg *.toc *.ps *.pdf

distclean:: clean
	rm $(TARGET).ps

.PHONY: clean all fresh distclean nobib
