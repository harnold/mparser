===================================================
mParser, a simple monadic parser combinator library
===================================================

This library implements a rather complete and efficient monadic parser
combinator library similar to the Parsec library for Haskell by Daan Leijen
and the FParsec library for FSharp by Stephan Tolksdorf.

See the file INSTALL.txt for building and installation instructions.
See the file LICENSE.txt for copying conditions.

Home page: https://bitbucket.org/cakeplus/mparser


mParser used to be a part of ocaml-base, a collection of useful OCaml
libraries by Holger Arnold [1]_.

It is possible to use it with pa_monad [2]_.

The only dependency is the PCRE-OCaml library [3]_.


Usage example
-------------

Let's implement a simple expression evaluator.

To save the typing effort, it is often handy to open the ``MParser`` module::

  open MParser


First, we define a parsing combinator ``expr``, which handles expression
parsing, taking care of the operator predecency issues::

  let infix p o =
    Infix (p |>> (fun _ a b -> (`Binop (o, a, b))), Assoc_left)

  let operators =
    [ [ infix (char '*') `Mul;
        infix (char '/') `Div ];
      [ infix (char '+') `Add;
        infix (char '-') `Sub ] ]

  let expr =
    expression operators (Tokens.decimal |>> fun i -> `Int i)


Next, we implement an interpreter for our expression tree::

  let rec calc = function
    | `Int i -> i
    | `Binop (op, a, b) ->
        match op with
          | `Add -> calc a + calc b
          | `Sub -> calc a - calc b
          | `Mul -> calc a * calc b
          | `Div -> calc a / calc b


The evaluator function::

  let eval (s: string) : int =
    match MParser.parse_string expr s () with
      | Success e -> calc e
      | Failed (msg, e) -> failwith msg


Using it::

  eval "4*4+10/2"  ->  21


Have fun!


References
----------

.. [1] ocaml-base homepage
       http://www.holgerarnold.net/software

.. [2] Syntax extension for Monads in OCaml
       http://www.cas.mcmaster.ca/~carette/pa_monad/

.. [3] Markus Mottl's OCaml software (PCRE-OCaml)
       http://www.ocaml.info/home/ocaml_sources.html