This post is going to add a lot of value to your life; I'm going to
show you how to:

1.  Install LaTeX on OS X, without too much junk.
2.  Use org-mode on emacs along with some creature comforts
3.  Use animations inside a PDF. Yes, I'm serious.

# Installing LaTeX

Most folks that install LaTeX on OS X probably do it with the [MacTex](https://tug.org/mactex/)
package. This is probably fine for most people, but the trouble with
MacTex is the size of the install; Its huge! The last time I checked,
MacTex ended up using at least 1.5 **gigabytes**. That's a lot of wasted
space, especially for things like the LaTeX editor (since I do my
LaTeX work in emacs). So here are the steps I recommend to installing
LaTeX on OS X.

## Installing LaTeX on OS X

1.  If you already have MacTex installed, get rid of it. Its pretty
    simple, just follow the directions [here](https://tug.org/mactex/uninstalling.html).
2.  Install BasicTex. This gives you stuff like pdfLatex, aka the
    compiler and core LaTeX. You can get the package installer from
    [here](http://www.tug.org/mactex/morepackages.html).
3.  Download the Tex Live Utility from [here](https://github.com/amaxwell/tlutility). Tex Live Utility is your 
    new LaTeX best friend. It will let you update all packages and
    install new ones whenever you need them, and you will need them
    since you only installed BasicTex. We will use it later on in this
    post.

# Org Mode

I'm going to assume that you already know what emacs is, but I'll
explain org-mode. Org-mode is an amazing mode that lets your write
notes, presentation, and more. Side note, these posts are written in
org-mode. I write them in org-mode, then org-mode exports them to
HTML; its pretty neat and convenient.

Okay, back to org-mode. In org-mode, you can put some configurations,
things like:

```org-mode
#+AUTHOR: Edgar
#+LATEX_HEADER: \usepackage{lmodern}
#+OPTIONS: toc:nil
```

Writing those things out on each new `.org` file is
annoying. Fortunately emacs already comes with a solution. You'll want
to add this to your `init.el`

```emacs-lisp
;; This turns on auto-insert-mode
(auto-insert-mode)
;; This turns off the prompt that auto-insert-mode asks before 
;; it actually inserts text/code for you
(setq auto-insert-query nil)
;; This is what you'll have inserted for a new .org file
(define-skeleton my-org-defaults
  "Org defaults I use"
  nil
  "#+AUTHOR:   Your name\n"
  "#+EMAIL:    your.email@gmail.com\n"
  "#+LANGUAGE: en\n"
  "#+LATEX_HEADER: \\usepackage{lmodern}\n"
  "#+LATEX_HEADER: \\usepackage[T1]{fontenc}\n"
  "#+OPTIONS:  toc:nil num:0\n")
;; This is how to tell auto-insert what to use for .org files
(define-auto-insert "\\.org\\'" 'my-org-defaults)
```

Now whenever you open a new `.org` file, you'll have these common
things already there in the buffer. You can do this for other modes
too, I do it for common `#include` items for C coding.

# Writing LaTeX via org-mode

You'll notice that in `my-org-defaults`, I included some items for
LaTeX. I used to write LaTeX in emacs by hand using Auctex and I
actually still recommend newcomers to LaTeX start by writing it by
hand. I use org-mode because the writing is so much easier in it, and
I can always drop down to LaTeX whenever I need to, like when I need
specific packages. For example, the `\usepackage{lmodern}` is a LaTeX
package for really nice modern latin font. To get this package, you'll
need to install it via the newly installed Tex Live Utility
application. Just open Tex Live Utility, go to the Packages tab,
search for lmodern and right click and install.

To export the `.org` file to PDF, just do: `C-c C-e l p`, where that
last `p` can be replaced with `o` to open the newly created
PDF. Another neat thing in org mode that I like is seeing LaTeX before
its a PDF, that is, I want to preview the LaTeX inside the
buffer. This, assuming you already have `imagemagick` installed, is
easily done with `C-c C-x C-l`.

Just to let you know my work-flow, I usually have a buffer open with
the PDF of my work and another buffer of my `.org` file. Then I export
to PDF and since I have `doc-view-mode` poll for updates, I see the
changes to my work nearly instantly in the PDF buffer.

# Animations in a PDF

For a final paper on modifications to the [Perceptron](http://en.wikipedia.org/wiki/Perceptron) algorithm, I
wanted to include a neat animation of the Perceptron
converging. Naturally I did my paper in emacs. Here are the steps
needed.

1.  Install the `animate` package using Tex Live Utility
2.  Add the following two lines to your `.org` file. 
    
    ```org-mode
    #+LATEX_HEADER: \usepackage{animate}
    #+LATEX_HEADER: \usepackage{graphics}
    ```

3.  For the place where you want to add the animation, add code like
    the following to your `.org` file:
    
    ```tex
    \begin{center}
    %                                            fps filename from to
      \animategraphics[autoplay, loop, width=4in]{2}{p_N200_it}{1}{68}
    \end{center}
    ```
    
    In this example, I gave it some optional arguments and said use 2
    frames per second and for all the filenames that start with
    p\_N200\_it, go from 1 to 68, for example `p_N200_it4.png`.
    
    **Note:** `\animategraphics` can take other optional arguments, like
    `controls`. Please sure to check out its easily googlable manual for
    all the details.

After exporting the PDF, you'll have a real animation that is
completely self contained in the PDF. 

**Note:** The Preview.app on OS X is kind of crappy and doesn't actually
animate the graphic for reasons unknown to me, I recommend you use
Adobe Reader.

I hope this post helps makes your papers all that more exciting and
rewarding.
