## NISIS autumn school 2019


These are instructions for the 
[NISIS autumn school 2019](https://nisis.sites.uu.nl/news-events/autumn-school-islamic-and-middle-eastern-studies-in-the-digital-age/),
workshop day Wednesday 2019-10-30, Utrecht.

It shows how a pipeline, suggested by
[Cornelis van Lit](https://digitalorientalist.com/about-cornelis-van-lit/),
from scanned book to OCR-ed book can be carried out
with open source tools.

### Prerequisites

??? abstract "macos"
    [Xcode](https://www.ics.uci.edu/~pattis/common/handouts/macmingweclipse/allexperimental/macxcodecommandlinetools.html)
    is required, but only the command line tools.

    ``` bash tab="macos"
    xcode-select --install
    ```

    We need a package manager to download, compile and install software.
    Such a package manager needs a C-compiler,
    which we installed as part of the command line tools of Xcode.

    There are two main options:
    [Homebrew](https://brew.sh)
    and
    [Macports](https://www.macports.org)

    We choose Macports, since it has all software we need.

    The install package can be downloaded from
    [Macports Installation](https://www.macports.org/install.php).
    You get a file `MacPorts-2.6.2-10.14-Mojave.pkg` or something similar.
    Double click to install.

    Then start the Terminal application, or restart it.

    From now on, all software installation takes place on the Terminal.

### Software

??? abstract "GhostScript"
    ``` bash tab="macos"
    sudo port install ghostscript
    ```

    [More info](https://www.ghostscript.com/doc/9.50/Readme.htm)

??? abstract "Imagemagick"
    ``` bash tab="macos"
    sudo port install ImageMagick
    ```

    [About](https://imagemagick.org/script/)

??? abstract "Scantailor"
    First we install the command line interface.

    ``` bash tab="macos"
    sudo port install scantailor
    ```

    [More info](https://github.com/scantailor/scantailor/wiki/User-Guide).

    But the fun is with the graphical interface.
    On the Mac, that is hard to get, we have to build from source.

    We have instructions to get that, but before that you may have to do this first

    ``` bash tab="macos"
    brew cask install xquartz
    ```
    
    [instructions that work](https://outofcontrol.ca/blog/installing-scantailor-on-macos-high-sierra).

??? abstract "Tesseract"
    We also need
    [language data](https://www.macports.org/ports.php?by=name&substr=tesseract-)
    for English.

    ``` bash tab="macos"
    sudo port install tesseract
    sudo port install tesseract-eng
    ```

    [More info](https://github.com/tesseract-ocr/tesseract)

??? abstract "Poppler"
    ``` bash tab="macos"
    sudo port install poppler
    ```

    [More info](https://poppler.freedesktop.org)

### Local organization

??? abstract "Working directory"
    Create a working directory somewhere on your system, it does not matter where.
    For the sake of this guide, I assume `local/pdfprocessing`.
    We can do this from the terminal.

    ``` bash tab="macos"
    cd
    mkdir local
    cd local
    mkdir pdfprocessing
    cd pdfprocessing
    ```

### Data

You can download the scan of a book as test data: 
[testBook.pdf](https://drive.google.com/file/d/1wl5_tRMxQclwQjen31sM6Bf1EVkCKsWH/view?usp=drive_web).

Put it in your working directory, `local/pdfprocessing`.

### Pipeline

??? abstract "Convert to JPG, per double page"
    We need to create the directory where the images will land:

    ``` bash tab="macos"
    mkdir images
    ```

    Now we can use image magick to wade through the pages of the pdf:
    
    ``` bash tab="macos"
    magick convert -density 200 testBook.pdf -quality 100 images/image-%02d.jpg
    ```

    After completion you have 117 JPG images in your `images`<D-d>irectory,
    one for each double page.

??? abstract "Split the double pages into single pages"
    We'll use the command line to work with Scantailor.

    First we need to create a directory where Scantailor can dump its output,
    we give it the name `scout`.

    ``` bash tab="macos"
    mkdir scout
    ```

    Then we call Scantailor with standard options:

    ``` bash tab="macos"
    scantailor-cli images scout
    ```

    Now `scout` contains 234 TIFF images, one for each single page.

??? abstract "Recompose the pages into a new PDF"
    ``` bash tab="macos"
    magick convert "scout/*.tif" -quality 100 out.pdf
    ```

    Now you have an `out.pdf` file of 14.9 MB.

??? abstract "OCR the TIFF images"
    We use Tesseract to perform Optical Character Recognition.
    First we move to the directory with TIFF images,
    then we create OCR-ed PDFs next to them.

    ``` bash tab="macos"
    cd scout
    ```

    ??? caution "Try it out first"
        The following command translates file names from `image-xxx` to `page-xxx`.
        Depending on your machine, the exact code might be different.
        If this goes wrong, chances are that tesseract writes the output for each page
        to exactly the same location, so after a long wait we only have the results
        for the last page.

        It is better to try this out on a small set.

        Use your file browser to make a directory `scout2` with only a few TIFF files,
        and then instead go to `scout2`

        ``` bash tab="macos"
        cd ../scout2
        ```

        Then run the command below, and verify of you get a file `page-xxx` next to
        each `image-xxx`.

        ??? hint "If not, check the bash manual"
            ``` bash tab="macos"
            man bash
            ```

            and search for `/Parameter Expansion` (the `/` is the search command). 

    ``` bash tab="macos"
    for f in *.tif; do tesseract $f page${f:5} PDF; done
    ```

    Now you have a lot of new `page-xxx.tif.pdf` files in `scout`.

??? abstract "Combine the OCR-ed PDFs into a single PDF"
    We use Poppler for the recombination step
    (the command `pdfunite` below comes from the Poppler suite).

    We go back to the top of our working directory, one level up.

    ``` bash tab="macos"
    cd ..
    pdfunite scout/*.pdf finalBook.pdf
    ```

    I got an error: a file appears to be empty.
    I moved it to a new directory `damaged`:

    ``` bash tab="macos"
    mkdir damaged
    mv scout/page-80_1L.tif.pdf damaged
    pdfunite scout/*.pdf finalBook.pdf
    ```

    Now you have a `finalBook.pdf` file of 6.9 MB
