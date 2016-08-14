# BIANCA

## Collaborate

Edit the bianca.md file using whichever text editor you want (Vim, Sublime, Word). 
You can even edit it online, directly on github: https://github.com/MathieuNls/bianca/edit/master/bianca.md

The only thing that matter is to save it back in text format.

## Build the pdf

The pdf generation is based on [Pandoc](http://pandoc.org/). Pandoc transforms the markdown to latex and then, to pdf. 
Here's the command, assuming pandoc is installed on your system and that you are on the Bianca directory :

```bash
pandoc -s -S --filter pandoc-citeproc --number-sections --template="config/default.latex" -o bianca.md.pdf *.md
```
