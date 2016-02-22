**Note** This series initially appeared on my previous blog and I'm just
 porting it over to here with small stylistic changes. 

This post will share some configurations I did to setup emacs for
linux kernel coding. Some minor context, the following emacs setup
came as a result of my Operating Systems class fall 2014. Note, a lot
of what I will go over is due to the great tutorials and walkthroughs
made by the emacs community, especially by Tuhdo's [great post](http://tuhdo.github.io/c-ide.html).

# The tools

Since our homeworks were assigned on a Debian VM, the natural
beginning tool was emacs + tramp. One thing early annoyance was tramp
constantly asking for passwords, so I put this in my init.el

```emacs-lisp
(setq password-cache-expiry nil)
```

and that took care of that. The next was CEDET/semantic, this one took
quite a bit of effort...

# Semantic

Emacs comes with CEDET, a suite of mainly C, C-like languages
utlities, the most useful of which is semantic. Semantic provides
autocomplete that makes emacs provide IDE like code
completion. Initially, I tried to make the builtin CEDET work, but no
matter all my efforts, it just wouldn't work right. So I proceeded to
get the source for CEDET and installed it, was really easy. After
getting CEDET install, I added this to my init.el.

```emacs-lisp
(load-file "~/.emacs.d/cedet/cedet-devel-load.elc")
(require 'semantic/senator)
(require 'semantic/ia)
(require 'semantic/analyze/refs)
(require 'semantic/analyze/complete)
(require 'semantic/bovine/gcc)
(require 'semantic/mru-bookmark)
(require 'semantic)
(global-semantic-idle-scheduler-mode 1)
(global-semantic-decoration-mode 1)
(global-semantic-idle-summary-mode 1)
(global-semantic-stickyfunc-mode 1)
(global-semantic-idle-local-symbol-highlight-mode 1)
(global-semantic-mru-bookmark-mode 1)
(global-semanticdb-minor-mode 1)
(global-cedet-m3-minor-mode 1)
(semanticdb-enable-gnu-global-databases 'c-mode t)
(global-semantic-show-unmatched-syntax-mode t)
```

I also used `company-mode` as the front end for semantic, you'll need
to edit the variable `company-backends` and have the list include the
symbol `company-semantic`. Next came the biggest issue, semantic was
having problems finding the #include because Linux's source code
headers are of the form `#include <header.h>`, which makes semantic
search in system headers and that is just all wrong for when you're
coding Linux. The original solution I found was to use EDE, but EDE is
just too damn confusing to work plus and doesn't seem to give much
bang for buck. Instead I went with a kludge, but like all good
kludges, it works. I defined this function:

```emacs-lisp
(defun os-s ()
  "Only way to get semantic to play nicely with desired files,
   very strange, *remember* to add the trailing slash for directories."
  (interactive)
  (setq company-c-headers-path-system '("/ssh:os:/home/w4118/hmwk6-prog/flo-kernel/arch/arm/i
                                        "/ssh:os:/home/w4118/hmwk6-prog/flo-kernel/include/")
  (setq company-c-headers-path-user '(" /ssh:os:/home/w4118/hmwk6-prog/flo-kernel/include/"))
  (semantic-reset-system-include)
  (setq semantic-dependency-include-path '("/ssh:os:/home/w4118/hmwk6-prog/flo-kernel/kernel/
  (semantic-add-system-include "/ssh:os:/home/w4118/hmwk6-prog/flo-kernel/arch/arm/include/")
  (semantic-add-system-include "/ssh:os:/home/w4118/hmwk6-prog/flo-kernel/include/")
  (setq-default semantic-symref-tool 'global)
  ;;For tramp mainly 
  ;; (setq default-directory "/ssh:os:")
  ;;Doesn't seem to work since this is a function defined in the shell? 
  ;; (local-set-key (kbd "M-z") '(lambda () 
  ;;                            (shell-command "b")))
  (mapc (lambda (item)
          (add-to-list 'tramp-remote-path item))
        '("/home/w4118/utils/android-sdk-linux/tools"
          "/home/w4118/utils/android-sdk-linux/platform-tools"
          "/home/w4118/utils/arm-eabi-4.6/bin"
          "/home/w4118/utils/bin"
          "/home/w4118/utils/arm-2013.11/bin"))
  (setq compile-command (concat
                         "make -j8 ARCH=arm CROSS_COMPILE=/home/w4118/utils/arm-eabi-4.6/bin/
                         " -C /home/w4118/hmwk6-prog/flo-kernel")))
```

Basically I wipe away the values that might be defined for semantic
and impose the ones of interest to me. I call this function via
another function that I use as a helper, it mainly opens a file in the
VM and calls `os-s`. You'll notice some commented out code, the first
was some time wasted with tramp and the second was trying to use an
alias in the remote shell, hint: it didn't work. 

That's it for this post. [Part Two](http://hyegar.com/blog/2015/09/11/emacs-for-linux-coding-part-ii/) relates to gnu global/helm.
