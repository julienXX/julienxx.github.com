---
title:  "Fancy Rust development with Emacs"
date:   2016-05-14 15:10:00
description: "A small tutorial on the crates and packages you need."
tags: ["rust", "emacs"]
---

[Rustup](https://www.rustup.rs) beta was released this week. As a member of the Church of Emacs I needed to update my config a bit. This inspired me to write this post on how to setup your Emacs as a great Rust IDE.


# Installing Rust

Thanks to `rustup` installing Rust on your system is now even easier. First do:

{% highlight shell %}
λ curl https://sh.rustup.rs -sSf | sh
{% endhighlight %}

When prompted choose `1) Proceed with installation (default)` unless you have some special requirements.

`rustup` was previously named `multirust`. It will create a `~/.multirust` directory to store rust [toolchains](https://github.com/rust-lang-nursery/rustup.rs#toolchain-specification).

The binaries and tools will be installed in `~/.cargo`. Make sure that you have `~/.cargo/bin` in your PATH otherwise add it with:

{% highlight shell %}
export PATH="$HOME/.cargo/bin:$PATH"
{% endhighlight %}

By default your projects will use the stable toolchain (Rust 1.8 at the time of this writing).

`rustup` has a lot of functionalities which you can find more about on the github [page](https://github.com/rust-lang-nursery/rustup.rs).

# Crates and packages you'll need

For this part I assume you already have a working emacs and that you know how to install packages and organize your configuration.


## rust-mode

`rust-mode` handles syntax highlighting, indentation and various settings for you.

Just `M-x package-install rust-mode`.


## cargo.el

[Cargo](http://doc.crates.io/index.html) is Rust package manager. `cargo.el` is a minor mode which allows us to run cargo commands from emacs like:

- `C-c C-c C-b` to run `cargo build`
- `C-c C-c C-r` to run `cargo run`
- `C-c C-c C-t` to run `cargo test`

and many others.

As for installation, nothing fancy just `M-x package-install cargo` and add:

{% highlight emacs-lisp %}
(add-hook 'rust-mode-hook 'cargo-minor-mode)
{% endhighlight %}

![Cargo run](/assets/images/cargo-run.gif)


## rustfmt

[rustfmt](https://github.com/rust-lang-nursery/rustfmt) formats your code according to community style guidelines just like `gofmt` in the Go language world. It's automatically handled by rust-mode.

First install the `rustfmt` crate:

{% highlight shell %}
λ cargo install rustfmt
{% endhighlight %}

I use `C-c <tab>` a lot in my emacs to automatically indent the current buffer so it would be nice if the same key combination would run `rustfmt` which also fixes indentation. So I simply added:

{% highlight emacs-lisp %}
(add-hook 'rust-mode-hook
          (lambda ()
            (local-set-key (kbd "C-c <tab>") #'rust-format-buffer)))
{% endhighlight %}

![rustfmt](/assets/images/rustfmt.gif)

The `ìndent-buffer` function I mentioned if you ever need it:

{% highlight emacs-lisp %}
(defun indent-buffer ()
  "Indent current buffer according to major mode."
  (interactive)
  (indent-region (point-min) (point-max)))
{% endhighlight %}


## racer

[Racer](https://github.com/phildawes/racer) is a code completion and source code navigation tool for Rust.

Install the racer crate:

{% highlight shell %}
λ cargo install racer
{% endhighlight %}

Rust source code is needed for auto-completion so clone it somewhere:

{% highlight shell %}
λ git clone git@github.com:rust-lang/rust.git
{% endhighlight %}

To use racer within emacs, do a `M-x package-install racer` and set it up like:

{% highlight emacs-lisp %}
(setq racer-cmd "~/.cargo/bin/racer") ;; Rustup binaries PATH
(setq racer-rust-src-path "/Users/julien/Code/rust/src") ;; Rust source code PATH

(add-hook 'rust-mode-hook #'racer-mode)
(add-hook 'racer-mode-hook #'eldoc-mode)
(add-hook 'racer-mode-hook #'company-mode)
{% endhighlight %}

Note that `racer` relies on [company-mode](https://company-mode.github.io) so install it if you haven't.

![racer](/assets/images/racer.gif)

## flycheck-rust

[flycheck](http://www.flycheck.org/en/latest/) is my prefered solution for on the fly syntax checking.
It will compile your code in background and highlight the problematic parts.

Let's `M-x package-install flycheck-rust` and then set it up like:

{% highlight emacs-lisp %}
(add-hook 'flycheck-mode-hook #'flycheck-rust-setup)
{% endhighlight %}

![flycheck](/assets/images/flycheck.gif)


# Wrapping up

That's all, you now have some great Rust support in your emacs with syntax highlighting, cargo handling, code formatting, errors highlighting, code-completion and source navigation. Happy Rust hacking!
