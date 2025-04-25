After creating the figure in KeyNote, remove the black border (or just make it
white), and export as a PDF.  Next, use `pdfcrop` to remove all the extra
surrounding whitespae:

```
pdfcrop foo.pdf foo.pdf
```
