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

```commonlisp
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

```commonlisp
> (defun many (n)
      (values n (* n 2) (* n 3)))

> (multiple-value-list (many 2))
(2 4 6)

> (nth-value 1 (many 2))
4
```

We can also use `multiple-value-bind` to assign each return value to a variable:

```commonlisp
> (multiple-value-bind (first second third)
      (many 2)
    (list first second third))
(2 4 6)
```

---

# Local variables

Local variables behave like in any other language: they are normal lexically scoped variables

Variables are declared with the `let` special operator:

```commonlisp
(let ((str "Hello, world!"))
    (string-upcase str))

;; => "HELLO, WORLD!"
```

You can define multiple variables:

```commonlisp
(let ((x 1)
      (y 5))
  (+ x y))

;; => 6
```

To define variables whose initial values depend on previous variables in the same form, use `let*`:

```commonlisp
(let* ((x 1)
       (y (+ x 1)))
    y)

;; => 2
```

---

# Dynamic Variables

Dynamic variables are sort of like global variables, but more useful: they are dynamically scoped

You define them either with `defvar` or `defparameter`, the differences being:

1. `defparameter` requires an initial value, `defvar` does not
2. `defparameter` variables are changed when code is reloaded with a new initial value, `defvar`
   variables are not

What does dynamic scoping mean? It means:

```commonlisp
(defparameter *string* "I'm global")

(defun print-variable ()
    (print *string*))

(print-variable) ;; Prints "I'm global"

(let ((*string* "I have dynamic extent")) ;; Binds *string* to a new value
    (print-variable)) ;; Prints "I have dynamic extent"
;; The old value is restored

(print-variable) ;; Prints "I'm global"
```

In other words, when you redefine the value of a dynamic variable using `let`, the variable is bound
to the new value inside the body of the `let`, and the old value is `restored` afterwards

---

# Lists

# Basics

Lists can be built using the `list` function:

```commonlisp
(list 1 2 3)
;; (1 2 3)
```

You can use `first`, `second`, and all the way up to `tenth` to access the corresponding elements of
a list:

```commonlisp
(fist (list 1 2 3))
;; 1

(second (list 1 2 3))
;; 2
```

These can also be used to set elements:

```commonlisp
(defparameter my-list (list 1 2 3))
;; MY-LIST

(setf (second my-list) 7)
;; 7

my-list
;; (1 7 3)
```

More generally, the `nth` function can be used:

```commonlisp
(nth 1 (list 1 2 3))
;; 2
```

And it works with `setf`:

```commonlisp
(defparameter my-list (list 1 2 3))
;; MY-LIST

(setf (second my-list) 65)
;; 65

my-list
;; (1 65 3)
```

# Higher-Order Functions

## Map

The `map` function takes a function and a list, goes through each element in the sequence, and
returns a new list where every element is the result of calling that function with the original element

For instance:

```commonlisp
(mapcar 'evenp (list 1 2 3 4 5 6))
;; (NIL T NIL T NIL T)
```

Is equivalent to:

```commonlisp
(list (evenp 1) (evenp 2) (evenp 3) (evenp 4) (evenp 5) (evenp 6))
;; (NIL T NIL T NIL T)
```

Another example:

```commonlisp
(mapcar #'string-upcase (list "Hello" "world!"))
;; ("HELLO" "WORLD!")
```

One way to help understand `mapcar` is by writing our own:

```commonlisp
(defun my-map (function list)
    (if list
        (cons (funcall function (first list))
              (my-map function  (rest list)))
        nil))
;; MY-MAP

(my-map #'string-upcase (list "a" "b" "c"))
;; ("A" "B" "C")
```

# Reduce

The `reduce function can be used to turn a list into a scalar, by applying a function on successive
subsets of the list. For instance:

```commonlisp
(reduce #'+ (list 1 2 3))
;; 6
```

You can also use a custom function:

```commonlisp
(reduce #'(lambda (a b)
            (* a b)
        (list 10 20 30))
;; 6000
```

The above is equivalent to `(* (* 10 20) 30)`.

To get a better understanding of how reduce works, we can use `format`:

```commonlisp
(reduce #'(lambda (a b)
            (format t "A: ~A, B: ~A~%" a b)
            (* a b))
        (list 1 2 3 4 5 6))

;; A: 1, B: 2
;; A: 2, B: 3
;; A: 6, B: 4
;; A: 24, B: 5
;; A: 120, B: 6
;; 720
```

# Sorting

The `sort` function allows you to sort a sequence:

```commonlisp
(sort (list 9 2 4 7 3 0 8) #'<)
;; (0 2 3 4 7 8 9)
```

# Destructuring

```commonlisp
(defun destructure (list)
    (destructuring-bind (first second &rest others)
     list
     (format t "First: ~A~%" first)
     (format t "Second: ~A~%" secont)
     (format t "Rest: ~A~%" others)))
```

This produces

```commonlisp
(destructure (list 1 2 3 4 5 6))
;; First: 1
;; Second: 2
;; Rest: (3 4 5 6)
;; NIL
```
---

# I/O

# Format

The most common way to print to the screen is using the `format` function

This is like C's `printf`, but embedding a whole language for printing

This [chapter](http://www.gigamonkeys.com/book/a-few-format-recipes.html) of [Practical Common Lisp](https://lisp-lang.org/books/#practical-common-lisp) contains a lot of useful `format` directives

## File I/O

You can use `with-open-file` to safely handle file

For instance, here's how we open a file `data.txt` in your home directory for writing:

```commonlisp
(with-open-file (stream (merge-pathnames #p"data.txt"
                                         (user-homedir-pathname))
                        :direction :output      ;; Write to disk
                        :if-exists :supersede   ;; Overwrite the file
                        :if-does-not-exist :create)
    (dotime (i 100)
      ;; Write random numbers to the file
      (format stream "~3,3f~%" (random 100))))
```

You can read the file into a string using `uiop:read-file-string`:

```commonlisp
(uiop:read-file-string (merge-pathnames #p"data.txt"
                                        (user-homedir-pathname)))
;; "44.000
;; 95.000
;; 5.000
;; 97.000
;; ...
;; 15.000"
```

---

# Macros

As an example, Common Lisp has no `while` loop, rather, there's a `loop` macro directive for
iterating while a conditionis true

For brevity, we can define:

```commonlisp
(defmacro while (condition &body body)
    `(loop while ,condition do (progn ,@body)))
```

[Pick up here](https://lisp-lang.org/learn/macros)
