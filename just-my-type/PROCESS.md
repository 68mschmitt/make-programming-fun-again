Documenting my process and thoughts
===

Thoughts on design (semantics)
- Terminal program
    - CLI at first (Easy one shot execution)
        - Run program, then type out something
    - TUI later
        - Take over the terminal to do typing stuff
- What am I going to type?
    - Could be the contents of a file
        - cat out the contents of a file, then just show the text in a semi-grey color
        - As you type out the text, correctly, the text turns white
        - If you make a mistake, then the text turns red
- Time limit
- Track metrics

---

- Minimal implementation
    - CLI command
    - input is a file name
    - Hard code a limit on the characters that need to be typed

---

Technical details
- LISP
    - I need to figure out how to set that up
    - I can do racket, but a repl seems to be the lisp way
    - emacs is the known lisp editor and repl, but I don't know for now

---

Getting started

I am going to work with common lisp

[Common lisp](https://lisp-lang.org/learn/)

I will go through some of the first steps and get a quickstart to get going

---

# Learn common lisp

install `sbcl` Steel Bank Common Lisp
`sudo pacman -S sbcl`

Install Quicklisp (package manager)
```sh
curl -o /tmp/ql.lisp http://beta.quicklisp.org/quicklisp.lisp
sbcl --no-sysinit --no-userinit --load /tmp/ql.lisp \
     --eval '(quicklisp-quickstart:install :path "~/.quicklisp")' \
     --eval '(ql:add-to-init-file)' \
     --quit
```

Install Emacs and SLIME

SLIME is a common lisp IDE built on emacs
You can install it with quicklisp using
`sbcl --eval '(ql:quickload :quicklisp-slime-helper)' -quit`

then, add it to your ~/.emacs.d/init.el:

```sbcl
(load (expand-file-name "~/.quicklisp/slime-helper.el"))
(setq inferior-lisp-program "sbcl")
```

# Running slime

You can run slime in emacs by pressing Alt+x then type slime and hit enter

---

# Fist steps

`(format t "Hello, world!")`

---

# Functions

```lisp
(defun fib (n)
    "Return the nth Fibonacci number."
    (if (< n 2)
        n
        (+ (fib (- n 1))
           (fib (- n 2)))))
```

Then call the function like any other
`(fib 30)`

---

# Anonymous Functions

## Application
Functions can be called indirectly using funcall:

`(funcall #'fib 30)`

or with apply:

`(apply #'fib (list 30))`

---

# Multiple Return Values

```lisp
> (defun many (n)
      (values n (* n 2) (* n 3)))

> (multiple-value-list (many 2))
(2 4 6)

> (nth-value 1 (many 2))
4
```

We can also use `multiple-value-bind` to assign each return value to a variable:

```lisp
> (multiple-value-bind (first second third)
      (many 2)
    (list first second third))
(2 4 6)
```

---

# Local variables


