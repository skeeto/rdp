# Recursive Descent Parser for Emacs Lisp

This package provides a recursive descent parser for Elisp
programs. Grammars are specified using a pure Elisp s-expression. A
set of matching functions can be provided to the parser to manipulate
the parse tree at read time. These functions can even be used to
[implement a compiler](https://github.com/skeeto/psl-mode/blob/master/psl-compile.el).

The parser will indicate where the point is in the parse tree, for use
in automatic indentation for major modes. This is no silver bullet:
automatic indentation is
[still a black art](http://www.gnu.org/software/emacs/manual/html_node/elisp/Auto_002dIndentation.html)].
SMIE is probably better suited if that's your goal -- to implement
indentation but not fully parse a language as rdp would.

Full documentation can be found in the[commentary section of
`rdp.el`](https://github.com/skeeto/rdp/blob/master/rdp.el).

## Example

This is an interpreter for simple arithmetic expressions, including
operator precedence and grouping.

```el
(defvar arith-tokens
  '((sum       prod  [([+ -] sum)  no-sum])
    (prod      value [([* /] prod) no-prod])
    (num     . "-?[0-9]+\\(\\.[0-9]*\\)?")
    (+       . "\\+")
    (-       . "-")
    (*       . "\\*")
    (/       . "/")
    (pexpr     "(" [sum prod num pexpr] ")")
    (value   . [pexpr num])
    (no-prod . "")
    (no-sum  . "")))

(defun arith-op (expr)
  (destructuring-bind (a (op b)) expr
    (funcall op a b)))

(defvar arith-funcs
  `((sum     . ,#'arith-op)
    (prod    . ,#'arith-op)
    (num     . ,#'string-to-number)
    (+       . ,#'intern)
    (-       . ,#'intern)
    (*       . ,#'intern)
    (/       . ,#'intern)
    (pexpr   . ,#'cadr)
    (value   . ,#'identity)
    (no-prod . ,(lambda (e) '(* 1)))
    (no-sum  . ,(lambda (e) '(+ 0)))))

(defun arith (string)
  (rdp-parse-string string arith-tokens arith-funcs))
```

Usage:

```el
(arith "(1 + 2 + 3 * 4)*-3/4.0")
```

This evaluates to -11.25, as would be expected.

## See Also

 * [SMIE](http://www.gnu.org/software/emacs/manual/html_node/elisp/SMIE.html)
 * [peg.el](http://emacswiki.org/emacs/peg.el)
 * [Semantic Bovinator](http://emacswiki.org/emacs/SemanticBovinator)
