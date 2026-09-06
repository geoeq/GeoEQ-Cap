# User guide sources

`user-guide.tex` builds `user-guide.pdf` with any TeX distribution that has
pdflatex (MiKTeX, TeX Live):

    latexmk -pdf user-guide.tex

Every screenshot in `figures/` is rendered from the application's bundled
demo project, so the pictures always show what the shipped build draws.
