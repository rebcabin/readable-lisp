This tutorial briefly describes our approach to making Lisp s-expressions more readable, and shows how to download and use free-libre/open source software that implements them.

Our approach is not tied to any particular Lisp system, and can do everything traditional s-expressions can do. But it's easier to try it out focusing on some specific Lisp; this tutorial focuses on the Common Lisp family (you can also look at the [Scheme-tutorial]).

You don't need to be familiar with Lisp-like languages to understand this tutorial, though it helps.

But first, let's briefly describe the problem and the proposed solution (feel free to skip these sections if you already know this).


What's wrong with Lisp and s-expressions?
=========================================

Lisp-derived systems normally represent programs as *s-expressions*, where an operation and its parameters is surrounded by parentheses. The operation to be performed is identified first, and each parameter afterwards is separated by whitespace. So the traditional “2+3” is written as “(+ 2 3)” in a Lisp-derived language. S-expressions make it unusually easy for programs to read, process, and generate programs. This can make some programs considerably easier to create, as well as much easier to automatically analyze or prove.

Unfortunately, programs in S-expression format can be hard to read. Lisp S-expressions fail to support infix notation, its notation for making function calls is completely different than most programming languages and math books, and it requires using and balancing endless parentheses. The last problem can be partly overcome through tools, but why use a notation that's so bad that such tools are important? It'd be better to have a notation that people can easily read in the first place.

For more information on the problem, see [Problem].


Proposed solution
=================

To solve this, we've developed three tiers of notation, each building on the previous. These are simply new abbreviations for common cases; you can continue to use normally-formatted s-expressions wherever you want to. These new notations/abbreviations are, in summary:

1.   *Curly-infix-expressions* (*c-expressions*): Curly braces {...} contain an *infix list*. A *simple infix list* has (1) an odd number of parameters, (2) at least 3 parameters, and (3) all even parameters are the same symbol; it maps to "(even-parameter odd-parameters)".  By intent, there is no precedence and you *must* use another {...} for an embedded infix list. For example, {n <= 2} maps to (<= n 2).  In "full" c-expressions, the items in an infix list are neoteric expressions.
2.   *Neoteric-expressions* (*n-expressions*): This includes curly-infix-expressions, and adds special meanings to some prefixed symbols. An e(...) maps to (e ...), and an e{...} maps to (e {...}). There must be no whitespace between e and the opening character.  For example, f(1 2) maps to (f 1 2)
3.   *Sweet-expressions* (*t-expressions*): Includes neoteric-expressions, and deduces parentheses from indentation. Basic rules:

    - A line with content consists of one or more n-expressions, separated by one or more spaces or tabs.
    - If a line is indented more than the previous line, that line is a child line, and the previous line is a parent to that child.
    - Later lines with the same indentation as the child are also children of that parent, until there is an intervening line with the parent’s indentation or less.
    - A line with only one n-expression, and no child lines, represents itself. Otherwise, the line represents a list; each n-expression on the line is an element of the list, and each of its child lines represents an element of the list (in order).
    - An empty line (possibly with spaces or tabs) ends the expression; empty lines *before* expressions are ignored.
    - Indentation processing does not occur inside ( ), [ ], and { }, whether they are prefixed or not; they're just neoteric-expressions.

For more details, see [Solution].

For example, here's some traditionally-formatted code:

    (define (factorial n)
      (if (<= n 1)
        1
        (* n (factorial (- n 1)))))

And the same thing with sweet-expressions:

    define factorial(n)
      if {n <= 1}
        1
        {n * factorial{n - 1}}

You can see many more examples in [Examples].


Getting our implementations
===========================

We've created implementations of curly infix, neoteric-expressions, and sweet-expressions as a Common Lisp library called "readable".

To try the Common Lisp demos, make sure you have a Common Lisp implementation.  We primarily use clisp and SBCL, but it should port to any implementation.  (If it doesn't, please help us fix that!)  For tutorial purposes, we'll focus on using clisp.

If you plan to use the development (not stable) version, you'll also need:

*   git (to download the software)
*   autoconf and automake (to build it)

You should be running on a POSIX system; if you use Windows, you may need to install Cygwin first. Any modern GNU/Linux system will do nicely.

Download and prepare to configure
-----------------------------------

First, download the latest version of the demo and related files and get them ready to configure.

At the time of this writing, the "stable" version doesn't strongly support Common Lisp.  Eventually it will.  To get the current "stable" version, browse http://readable.sourceforge.net - click on "download files" - and download the current version.  Then uncompress and cd into them, e.g., at the command line:

     tar xvzf readable-*.tar.gz
     cd readable-*

If you *instead* want to get the *development* version (as you should do for now), do this instead:

     git clone git://git.code.sf.net/p/readable/code readable-code
     cd readable-code
     git checkout develop  # Switch to "develop" branch
     autoreconf -i

If you use one of the rare systems that doesn't support /usr/bin/env, then you'll need to install one ("ln -s /bin/env /usr/bin/env") or change the first lines of some of the scripts.

Configure
---------

As usual, configure.  To accept all the defaults, installing to /usr/local:

    ./configure

If you want it to install to /usr (e.g., programs go in /usr/bin):

    ./configure --prefix=/usr

It will probably be easier for you if you install in /usr.

Build
------

Now build the code, using:

    make

Install
-------

You actually don't have to install it to use it, if you just want to play with it.  But if you're using this software seriously, you should probably install it to "real" locations.  If you do, just type:

    make install

This will probably require privileges, so you may need to run "su" first or use sudo ("sudo make install") to get those privileges.

It's a lot easier to use if you install it, but if you have not (or can't), the text below will explain how to work around that.


Starting Common Lisp implementation and loading the readable library
====================================================================

You need to run your Common Lisp implementation and load the "readable" library.

If you've installed the "readable" library (as discussed above), and you already have the standard "ASDF" library system installed, this is easy. Here's how you do it for clisp at the command line:

    clisp  # Run the "clisp" program; feel free to add the "-modern" option
    (require 'asdf)  ; Load "adsf", the standard system for loading libraries
    (asdf:load-system :readable)  ; Load the "readable" library

Replace "clisp" with whatever Common Lisp you want (e.g., sbcl).

If you don't have asdf installed, well, you need to get it.  This is particularly an issue with Cygwin, where clisp doesn't come with asdf.  Just download asdf (from http://common-lisp.net/project/asdf/ ) - and if you don't want to formally install it, you can put the .lisp file somewhere, and instead of require "asdf", use:

    (load "/your/path/to/asdf.lisp")

If you don't want to install the "readable" library in the standard central place, you can work around that too, but you need to give both asdf and the underlying Lisp information on where to load it from.  If your current directory ($PWD) contains the readable .lisp files, you can replace the "clisp" command with the following to do this:

    CL_SOURCE_REGISTRY="$PWD" clisp -lp "$PWD"


After you've loaded the readable library, you can use it.  The instructions below assume that you don't "use" the package, so you'll have to include the name of the package to invoke anything in it.  If you find that inconvenient, you can omit all the "readable:" prefixes by first using the standard Common Lisp "use-package" command:

    (use-package :readable)


Using basic curly-infix-expressions (c-expressions)
=============================================

Let's first try out "curly-infix-expressions" (c-expressions).  Let's start by enabling basic c-expressions, which will modify our existing readtable to add these capabilities:

    (readable:enable-basic-curly)

Any of these three new notations (curly-infix-expressions, neoteric-expressions, and sweet-expressions) can also accept normally-formatted s-expressions. To demonstrate that, type at the command line:

     (+ 2 3)

and when you press enter you'll see 5.

Basic curly-infix-expressions add the ability to use {...} around infix operators.  Basically, {...} also creates a list, but in the usual case the reader converts the *even* parameter(s) into the *first* parameter.  So just type this in:

    {2 + 3}

and you will see 5. Look! You now have a calculator! It's important to realize this is a simple syntactic convenience — an abbreviation automatically handled by the system when it reads the input. Traditional Lisp actually includes many abbreviations, for example, 'x is a traditional abbreviation for (quote x), which when executed will just return the quoted x.  So curly-infix-expressions just includes an additional abbreviation and does not change how "Lisp works". Thus, if you enter:

    '{2 + 3}

it will respond with (+ 2 3).  That is, once it's read in, it will read in a quoted (+&nbsp;2&nbsp;3), and "executing" it will return the simple (+&nbsp;2&nbsp;3).  Note that the infix operator *must* be surrounded by whitespace - otherwise, it would have no way to know where the name of the operation begins or ends.

There is intentionally no support for precedence between different operators. While precedence is useful in some circumstances, in typical uses for Lisp-derived languages and sweet-expressions, it doesn't help and is often harmful. In particular, precedence can hide where different lists occur.  This lack of precedence is not a problem at all; usually you can just use curly braces or parentheses when mixing different infix operators:

    {2 + {3 * 4}}

You *can* "chain" the same infix operator, e.g., {2 + 3 + 4} is fine, and it will map to (+ 2 3 4). This works with comparisons, too, as long as the comparisons are the same.  Here's an example:

    {5 <= 7 <= 10}

In short, a *simple infix list* has (1) an odd number of parameters, (2) at least 3 parameters, and (3) all even parameters are the same symbol; it maps to "(even-parameter odd-parameters)".  By intent, there is no precedence and you *must* use another {...} for an embedded infix list.

If a curly-infix list has 0, 1, or 2 parameters, they are treated specially.  {} is a synonym for (), since they are both empty lists.  { e } maps to just e, and { - x } maps to (- x).

But what happens to infix lists that aren't any of these?  The answer is that they map to "($nfx$ parameters)".   You can then define a macro named "$nfx$" to process the list (e.g., to implement a precedence system), or when processing lists, note this symbol specially.  The dollar signs make it unlikely that you'd define or use this by accident.

Note that curly-infix-expressions intentionally do not force a particular precedence, nor do they automatically switch to infix operators recursively inside {...}. Many previous systems did that, but this turned out to interfere with many of Lisp's power (which needs homoiconicity and generality). It also does not attempt to guess when infix operators are used. [After many experiments with real problems, David A. Wheeler found that this rule works better for Lisps than those alternatives.](http://www.dwheeler.com/readable/version02.html)

The main advantage of basic c-expressions is that they're really simple in concept, and very simple to implement.  They have almost no chance of conflicting with anything else; they only redefine "{" and "}" in the readtable, for example.  In particular, that's all there is to basic c-expressions, they're really easy.

To disable basic curly-expressions (or any other readable notation) just run this, which restores the previous readtable:

    (readable:disable-readable)


Using full curly-infix
======================

You can enable full curly-infix by running:

    (readable:enable-full-curly-infix)

In full curly-infix, the list elements inside {...} are (recursively) neoteric-expressions... but only inside some {...}. 

In neoteric-expressions, an expression of the form f(...) is treated the same as (f ...).  Thus, inside a curly-infix list, cos(0) is treated the same as (cos 0), which evaluates to 1.  So we can do this:

    {cos(0) + 1}

We can explain more about neoteric-expressions... but let's learn more by disabling full-curly-infix, and switching to neoteric-expressions entirely:

    (readable:disable-readable)


Using neoteric-expressions (n-expressions)
==========================================

You can enable neoteric-expressions by running:

    (readable:enable-neoteric)

The difference between enable-neoteric and enable-full-curly-infix is that enable-neoteric lets you type in neoteric-expressions at the topmost level.  With enable-full-curly-infix, you have to surround neoteric-expressions with {...}.

Neoteric-expressions support curly-infix-expressions, including normally-formatted s-expressions.  In addition, neoteric-expressions add special meanings to the grouping symbols ( ), [ ], and { } if they *immediately* follow an expression (instead of being separated by whitespace).  In particular, any e( ... ) is mapped to (e ...), and any e{ ... } is mapped to (e { ... }).

This means that you can use more "traditional" functional notation, e.g., f(1 2) maps to (f 1 2). Just type in the name of a function, an opening "(", its parameters (space-separated), and a closing ")". Make sure that you do *not* have a space before the (prefixed) function name and the following "(". For example, type this in:

     cos(0)

and get a very reasonable response, 1. Here's another - try this, to use subseq to get a substring:

    subseq("Hello" 1 3)

This will produce "el".

You can nest them, just as you'd expect:

    subseq("Hello" 1 length("xyz"))

Using function name prefixes is a nice way of showing negation, e.g., -(x) computes the value of 0 - x. So while curly infix by itself doesn't handle prefix functions, neoteric-expressions can handle them nicely:

    defun(neghalf (n)
        {-(n) / 2})

You can even use function name prefixes with traditional binary operators, such as:

    *(5 4)

This works with zero parameters, too; if you have a command called "help" (guile does), and choose not to give it any parameters, just type this (without pressing space before typing it in):

    help()

It's actually quite common to have a function call pass one parameter, where the parameter is calculated using infix notation, so there's a rule to simplify this common case. You can use f{x + 1}, which maps to (f {x + 1}) which then maps to (f (+ x 1)). This makes it easy to pass a single parameter which happens to be calculated using infix. For example, factorial{n - 1} maps to factorial({n - 1}) which maps to (factorial (- n 1)).  You can try out this simple example:

     not{t and nil}

Just like traditional s-expressions, spaces separate parameters, so it's *important* that there be *no* space between the function name and the opening "(". Since spaces separate parameters, a space between the function name and the opening "(" would create two parameters instead of a single function call. The same is basically true for traditional s-expressions, too; (a b) and (ab) are not the same thing.

Here's the real rule: in neoteric-expressions, e(...) maps to (e ...), e{} maps to (e), other e{...} maps to (e {...}), e[ ... ] maps to ($bracket-apply$ e), and (. e) maps to e. The "$bracket-apply$" is so that you can write a macro to access arrays and other mappings.  The (. e) rule lets you escape expressions (e.g., for the sweet-expressions we'll describe next).  Note that "neoteric-expressions" used be called "modern-expressions"; you may see some older documents using that name.

The advantage of full-curly-infix is that it is totally and completely compatible with traditional s-expressions (since traditional s-expressions do not define what happens inside {...} anyway).  Enable-neoteric can, in extremely unlikely scenarios, cause expressions to be interpreted differently... but they let you use n-expressions directly.

Normally, people and pretty-printers will format Lisp code so that parameters inside a list are *separated* by whitespace, e.g., (a b c), so it turns out that this change in interpretation doesn't change the meaning of typically-formatted modern Lisp code (and you can pretty-print code to fix it in the rare cases you need to). What's more, typical Lisp-aware text editors can often work with neoteric-expressions as they are, without change... so if you don't want to change the way you work, but have a somewhat more readable notation, neoteric-expressions can help.  If you're worried about some expression being interpreted differently, you can disable neoteric-expressions while you read them... or just run them through a pretty-printer (pretty-printers fix such weird formats normally anyway).

Neoteric-expressions are a nice step forward.  But we still have to do all that parentheses-balancing, which hinders readability. Sweet-expressions, our next stop, address this.

To end this part of the demo, disable our changed notation:

    (readable:disable-readable)


Using Sweet-expressions (t-expressions) 
=======================================

Sweet-expressions take neoteric-expressions and infers parentheses from indentation.  You can enable neoteric-expressions by running:

    (readable:enable-neoteric)

In sweet-expressions:

- An indented line is a parameter of its parent, and later terms on a line are parameters of the first term.
- A line with exactly one term, and no child lines, is simply that item; otherwise those terms and its child lines are themselves a new list.
- Lines with *only* a ;-comment, and nothing else, are completely ignored - even their indentation is irrelevant.
- Whitespace-only lines at the beginning of a new expression are ignored, but a whitespace-only line (including a zero-length line) ends an expression.

So, just type in your expression, and type a blank line (an extra Enter) to indicate that you're done.

Here's a trivial example; type this in, and enter a blank line (*Enter* *Enter*) to calculate it:

    subseq "Hello"
      1
      3


Be sure to type *Enter* *Enter* (a blank line) to execute the expression.

What happens if the parameters are not constants, but something to be calculated? No problem, just put them on new lines and give them parameters. If something has parameters, then it must be something to calculate too! Here's another example (be sure to consistently indent the lines after the first one):

    subseq
      "Hello"
      1
      string-length "xyz"

You can use parentheses, too; inside any grouping characters (...), [...], and {...}, indentation is ignored:

      substring
        "Hello"
        1
        length("xyz")

Here are some other valid sweet-expressions:

    if {7 < 5}
      {3 + 4}
      {5 * {2 + 3}}
    
    abs{0 - 5}

Here's a more substantial example:

    defun fibfast (n)
      if {n < 2}       ; Indentation, infix {...}
        n              ; Single expr = no new list
        fibup n 2 1 0  ; Simple function calls
    
    defun fibup (max count n1 n2)
      if {max = count}
        {n1 + n2}
        fibup max {count + 1} {n1 + n2} n1

If you're not sure what something means, you can "quote" it so it won't execute.  If you type ' followed by space, the indentation processing continues (starting at the ' mark indentation) but the whole thing will be quoted.  That way you can try things out!  For example:

    ' foo bar1 bar2
        spam eggs eggs

Will produce:
(foo bar1 bar2 (spam eggs eggs))

Sometimes you want to have a parameter that is a list of lists, or where the function to be called is in fact determined by another calculation. This is indicated with the "\\\\" keyword; basically, at the beginning of line (but after indentation) "\\\\" maps into a null function name, so you can use forms like "let" easily.


Sweeten: Translating S-expressions into Sweet-expressions
=========================================================

We also have a filter called "sweeten" that translates existing S-expressions into sweet-expressions (this is the *reverse* of unsweeten).  You can use sweeten to help you understand sweet-expressions, or to move an existing program to sweet-expressions.

Just put some S-expressions into a file and you can have them translated.  For example, you could the following in file "demo.scm":

    (defun doubleadd (a b)
        (* (+ a b) 2))


You can run the sweetener by typing:

    ./sweeten -C < demo.scm

And you should see this as output:

    defun doubleadd (a b) {{a + b} * 2}

You can also run it interactively, but you may find that you want to press an extra "Enter" after your expression.  (It's done this way so that when it translates files they look correct.)

Sweeten is designed to read Scheme (or more specifically, Guile) S-expression syntax.  The "-C" option  adds some support for Common Lisp notation instead, in particular, #'function.  However, its handling of Common Lisp is far from perfect.  There is also a limitation of the sweeten implementation: Comments inside any parentheses will not be produced in the output.  (This is because it uses the underlying system "read" function, which throws this information away.) Still, it's a useful tool.

Readable library
=========================

The library comes with some other capabilities, e.g.:

*   curly-infix-read: Read a curly-infix-expression datum.
*   neoteric-read: Read a neoteric-expression datum.
*   sweet-read: Read a sweet-expression datum.


More Complex Examples
=====================

Basic Expressions
-----------------

If you're familiar with traditional s-expressions, here are more examples. The left-hand-side are sweet-expressions; the right-hand-side are the transitional s-expression forms, though the sweet-expression reader can read them as well:

    factorial(z)         <==> (factorial z)
    foo(x y)             <==> (foo x y)
    bar(x y z)           <==> (bar x y z)
    factorial{x - 1}     <==> (factorial {x - 1}) <==> (factorial (- x 1))
    f(g(y) h(y) a)       <==> (f (g y) (h y) a)
    f({x + 2} y {z - 3}) <==> (f (+ x 2) y (- z 3))
    f{{x + 2} * {y - 3}} <==> (f (* (+ x 2) (- y 3)))


Grouping and Splitting
----------------------

A basic indentation processor can sometimes require an excessive amount of vertical space, or it can make it hard to show the relationship between items at the same list level.  The \\\\ marker can help resolve this.

When a \\\\ marker is first on a line (after optional indentation), it stands for nothing at all.  This is useful for indicating groups of lists, e.g.,:

      let*
        \\
          asunit        unit(m)
          length-asunit length(asunit)
        if ...

=>

    (let* ((asunit        (unit m))
           (length-asunit (length asunit)))
      (if ...))


The \\\\ marker is also useful, when first on a line, to show that this line is logically subordinate to the previous line, but nevertheless it is at the same list level. For example:

    arc-if
      complicated-condition1()
      \\ do-if-complicated-condition1-is-true()
      complicated-condition2()
      \\ do-if-complicated-condition1-is-true()
      ...

=>

    (arc-if
      (complicated-condition1)
      (do-if-complicated-condition1-is-true)
      (complicated-condition2)
      (do-if-complicated-condition1-is-true)
      ...)

If you have small items that need to be at the same list level, you can combine them on one line and separate them with \\\\.  For example, this is useful for keyword symbols:

    foo
      :amount \\ 100 \\ :from \\ ocean

=>

    (foo :amount 100 :from ocean)

But what if you actually want to refer to "\\\\"?  No problem, just use the expression "(. \\\\)".  Exactly what this means will depend on whether or not your Lisp uses "slashification" (e.g., if "\" followed by any character means that character is part of a symbol).  If it doesn't use slashification, \\\\ means the symbol with a two-character name \\\\.  If it uses slashification, \\\\ means the symbol with the one-character name \\.  The good news is that you can use the marker \\\\ on practically any Lisp system, so you don't need to constantly change notation if you use different ones.

Sublist
-------

Sweet-expressions support another special abbreviation when not inside ( ... ), [ ... ], or { ... }.  A $ (aka SUBLIST) in the middle of list restarts list processing; the right-hand-side (including its sub-blocks) is the last parameter of the left-hand side.  This is really handy when you need to chain functions together, e.g., where a sequence of functions process in sequence the output of something more complex.  For example:

    onfail $ run $ head -10 "README"

has the same meaning as:

    (onfail (run (head -10 "README")))

Empty lines
-----------

Since empty lines end an expression, you can't have an empty line in the middle of one.  That's not as bad as it sounds, though.  You can instead use a semicolon comment (the indentation doesn't matter in this case) or \\\\ by itself at the correct indentation.

The reason is that this (1) makes interactive use pleasant, and (2) we don't want the interactive and file syntax to be different.

Collecting lists
----------------

When indentation is active, you can use <\* and \*\> as essentially parentheses, but indentation processing is still active, and inside the indent starts again at the edge.  This is useful for short expressions after a "let", as well as for creating long module definitions. E.G.:

    let <* x cos(a) *>
        {2 * x}

which becomes:

    (let ((x (cos a)))
        (* 2 x))


What's the big deal?
====================

If you aren't familiar with Lisp, you may say "what's the big deal"? After all, this looks a lot like traditional languages. Many have commented that it looks like Python, with its use of indenting, and of course nearly all other languages use infix notation.

But that's the point - the results look much more familiar (and thus are more acceptable to non-Lispers), but all of Lisp's more exotic capabilities still work. You can use techniques like quoting (') and quasi-quoting (\`) with lifting (,), which enable powerful capabilities. Many people have created "infix" notations with Lisp-like languages before, but they all failed to work with many other Lisp features. We think this approach succeeds instead, where others before have failed.


Closing Remarks
===============

These notations can take a few minutes to learn how to use, just like anything else new, but they are worth it.

Although we used some specific implementations, note that these notations could be used with any Lisp-based system.