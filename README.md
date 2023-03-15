# reason-mode
![Build Status](https://travis-ci.org/reasonml-editor/reason-mode.svg?branch=master)

An Emacs major mode for [ReasonML](https://reasonml.github.io/).

## Installation

### Prerequisites

**Note**: the following setup assumes Reason and Merlin are installed. This can be achieved by installing them from OPAM (`opam install reason merlin`). 

If you are using bucklescript, make sure you are using a compatible OCaml version (you can find the version of ocaml compatible with your bucklescript installation by running `npm bsc -version`).
At the time of writing this documentation, install OCaml 4.06.1 (for bucklescript 7.\*)

**Please verify your installation**:

```sh
ocamlc -version # 4.06.1 if you are using bucklescript
which ocamlmerlin # a valid path to the ocamlmerlin binary, mandatorily
which ocamlmerlin-reason # a valid path to the ocamlmerlin-reason binary, mandatorily
```

### MELPA

If your Emacs has `package.el` (which is automatically the case for Emacs >= 24), you can install `reason-mode` from the package in [MELPA](https://melpa.org/#/getting-started).

### QUELPA
Alternatively, you can use [quelpa](https://github.com/quelpa/quelpa) and the following recipe:

```lisp
(quelpa '(reason-mode :repo "reasonml-editor/reason-mode" :fetcher github :stable t))
```

### Manual Installation

Download `reason-indent.el`, `reason-interaction.el`, `reason-mode.el` and `refmt.el` at the root of this repository and place it in a `vendor` file next to your Emacs configuration files. Then place the following somewhere in your `.emacs.el`:

```lisp
(add-to-list 'load-path "/path/to/vendor")
```


Add the following to your `~/.emacs` or `~/.emacs.d/init.el` file:

```elisp
;;----------------------------------------------------------------------------
;; Reason setup
;;----------------------------------------------------------------------------

(defun shell-cmd (cmd)
  "Returns the stdout output of a shell command or nil if the command returned
   an error"
  (car (ignore-errors (apply 'process-lines (split-string cmd)))))

(defun reason-cmd-where (cmd)
  (let ((where (shell-cmd cmd)))
    (if (not (string-equal "unknown flag ----where" where))
      where)))

(let* ((refmt-bin (or (reason-cmd-where "refmt ----where")
                      (shell-cmd "which refmt")
                      (shell-cmd "which bsrefmt")))
       (merlin-bin (or (reason-cmd-where "ocamlmerlin ----where")
                       (shell-cmd "which ocamlmerlin")))
       (merlin-base-dir (when merlin-bin
                          (replace-regexp-in-string "bin/ocamlmerlin$" "" merlin-bin))))
  ;; Add merlin.el to the emacs load path and tell emacs where to find ocamlmerlin
  (when merlin-bin
    (add-to-list 'load-path (concat merlin-base-dir "share/emacs/site-lisp/"))
    (setq merlin-command merlin-bin))

  (when refmt-bin
    (setq refmt-command refmt-bin)))

(require 'reason-mode)
(require 'merlin)
(add-hook 'reason-mode-hook (lambda ()
                              (add-hook 'before-save-hook 'refmt-before-save)
                              (merlin-mode)))

(setq merlin-ac-setup t)
```

If you have iedit mode set up:

```lisp
(require 'merlin-iedit)
(defun evil-custom-merlin-iedit ()
  (interactive)
  (if iedit-mode (iedit-mode)
    (merlin-iedit-occurrences)))
(define-key merlin-mode-map (kbd "C-c C-e") 'evil-custom-merlin-iedit)
```

(Thanks @sgrove: [https://gist.github.com/sgrove/c9bdfed77f4da8db108dfb2c188f7baf](https://gist.github.com/sgrove/c9bdfed77f4da8db108dfb2c188f7baf))

This associates `reason-mode` with `.re` and `.rei` files. To enable it explicitly, do <kbd>M-x reason-mode</kbd>.

### Project specific version of `refmt`

If you're using different versions of `refmt` between projects, you can use the project-specific installed version via the special config values:
- `'npm` (calls `npx refmt ...` to use the version of `refmt` installed in the project's `node_modules`) 
- `'opam` (calls `opam exec -- refmt ...` to use the version of `refmt` on the current `opam` switch):

```lisp
;; can also be set via M-x `customize-mode`
(setq refmt-command 'npm)
```

### Utop

Reason-mode provides (opt-in) `rtop` support. At the moment only the native workflow is supported.

First of all you need to install the [Utop Emacs integration](https://github.com/diml/utop#integration-with-emacs). Make sure it is latest `master` because the feature is fairly new.

Then in your Emacs init file add:

```lisp
(require 'utop)
(setq utop-command "opam config exec -- rtop -emacs")
(add-hook 'reason-mode-hook #'utop-minor-mode) ;; can be included in the hook above as well
```

After this, the function `utop` (`C-c C-s`) will start `rtop` in Reason buffers.

### Spacemacs

The [`reasonml`](https://develop.spacemacs.org/layers/+lang/reasonml/README.html) layer is available in the develop version of spacemacs.


For the stable version of spacemacs, you can install the `reason-mode` package automatically.

```lisp
dotspacemacs-additional-packages
  '(
    (reason-mode
      :location (recipe
        :repo "reasonml-editor/reason-mode"
        :fetcher github
        :files ("reason-mode.el" "refmt.el" "reason-indent.el" "reason-interaction.el")))
)
```

Afterwards add the [snippet](#manual-installation) to your `dotspacemacs/user-config`.

### Features

#### Auto-format before saving

If you have refmt installed, you can add this to your `.emacs` file to enable
auto-format:

```lisp
(add-hook 'reason-mode-hook (lambda ()
          (add-hook 'before-save-hook #'refmt-before-save)))
```

### Tests via Cask + ERT

The `test` folder contains tests that can be run via [Cask](https://github.com/cask/cask).
Once you install `cask`, if it is the first time run:

```
cask install
cask exec ./run_tests.sh
```

If it is not the first time you can omit the first line and execute the tests with the second one only.
The environment variable EMACS controls the program that runs emacs.

## License

`reason-mode` is distributed under the terms of both the MIT license and the
Apache License (Version 2.0).

See [LICENSE-MIT](LICENSE-MIT) and [LICENSE-APACHE](LICENSE-APACHE) for details.
