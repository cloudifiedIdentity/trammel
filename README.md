Trammel
=======

[Contracts programming](http://c2.com/cgi/wiki?DesignByContract) for Clojure.

[![Clojars Project](http://clojars.org/trammel/latest-version.svg)](http://clojars.org/trammel)

- [Official documentation and usage scenarios](http://fogus.github.com/trammel/)
- [Original announcement](http://blog.fogus.me/2010/05/25/trammel-contracts-programming-for-clojure/) (*syntax has evolved since then*)

Example
-------

### Function Contracts

```clojure

    (require '[trammel.provide :as provide])
    
    (defn sqr [n] (* n n))
    
    (sqr 10)
    ;=> 100
    (sqr 0)
    ;=> 0
    
    (provide/contracts 
      [sqr "given a number not equal to zero, sqr ensures that it returns a positive number"
        [x] [number? (not= 0 x) => number? pos?]])
    
    (sqr 10)
    ;=> 100
    (sqr 0)
	; Pre-condition failure: given a number not equal to zero, sqr
    ;   ensures that it returns a positive number
    ; Assert failed: (not= 0 x)

```

### Record Invariants

```clojure

    (use '[trammel.core :only (defconstrainedrecord)])
    
    (defconstrainedrecord Foo [a b]
	  "Foo record fields are expected to hold only numbers."
      [(every? number? [a b])]
      Object
      (toString [this] (str "record Foo has " a " and " b)))
    
    ;; default ctor with default values
    (->Foo 1 2)
    ;=> #:user.Foo{:a 1, :b 2}
    
    ;; use like any other map/record
    (assoc (->Foo 1 2) :a 88 :c "foo")
    ;=> #:user.Foo{:a 88, :b 2, :c "foo"}
    
    ;; invariants on records checked at runtime    
    (assoc (->Foo 1 2) :a "foo")
	; Pre-condition failure: Foo record fields are expected to hold only numbers.
    ; Assert failed: (every? number? [a b])

```

### Type Invariants

```clojure

    (use '[trammel.core :only (defconstrainedtype)])
    
    (defconstrainedtype Foo [a b]
	  "Foo type fields are expected to hold only numbers."
      [(every? number? [a b])])
    
    (->Foo 1 2)
    #<Foo user.Foo@73683>
    
    ;; invariants on types checked at constructions time
    (->Foo 1 :b)
    ; Assert failed: (every? number? [a b])

```

### Reference Invariants

```clojure

    (def a (constrained-atom 0
         "only numbers allowed"
         [number?]))
    
    @a
	;=> 0
    
	(swap! a inc)
	;=> 1
	
    (swap! a str)
	; Pre-condition failure: only numbers allowed 
	
    (compare-and-set! a 0 "a")
	; Pre-condition failure: only numbers allowed 

```

The same will work on all reference types, including:

* **Refs**   - Invariants checked in a transaction
* **Agents** - Invariants checked on `send` and `send-off`, assertion errors handled as normal agent errors
* **Vars**   - Invariants checked on `binding`

Getting
-------

### Leiningen

Modify your [Leiningen](http://github.com/technomancy/leiningen) dependencies to include Trammel:

```clojure

    :dependencies [[trammel "0.7.0"] ...]

```

### Maven

Add the following to your `pom.xml` file:

```xml

    <dependency>
      <groupId>trammel</groupId>
      <artifactId>trammel</artifactId>
      <version>0.7.0</version>
    </dependency>

```

Notes
-----

Trammel is in its infancy but I think that I have a nice springboard for experimentation and expansion, including:

  - Contracts for higher-order functions
  - Better error messages
  - Distinct pre and post exceptions
  - Study the heck out of everything Bertrand Meyer and Walter Bright ever wrote (in progress)
  - `defconstraint` -- with ability to relax requires and tighten ensures
  - Study the heck out of Racket Scheme (in progress)
  - Modify macros to also allow regular Clojure constraint maps
  - Make the `anything` constraint cheap (elimination)
  - Allow other stand-alones: true/false, numbers, characters, regexes
  - Make `provide-contracts` more amenable to REPL use
  - Generate a Foo? function  (in progress) 
  - Marrying test.generative with Trammel

If you have any ideas or interesting references then I would be happy to discuss at me -the-at-sign- fogus -the-single-period- me.

References
----------

- [An Axiomatic Basis for Computer Programming](http://citeseer.ist.psu.edu/viewdoc/summary?doi=10.1.1.116.2392) by C.A.R Hoare -- essential reading
- *Object-oriented Software Construction* by Bertrand Meyer
- *Eiffel: The Language* by Bertrand Meyer
- [D](http://www.digitalmars.com/d/2.0/dbc.html)
- *The Fortress Language Specification* by Guy L. Steele Jr., et al.
- [Contracts for Higher-order functions](http://www.ccs.neu.edu/racket/pubs/NU-CCIS-02-05.pdf) by Robert Bruce Findler and Matthias Felleisen
- [System.Diagnostics.Contracts](http://msdn.microsoft.com/en-us/library/system.diagnostics.contracts.aspx)
- [Design by Contract and Unit Testing](http://onestepback.org/index.cgi/Tech/Programming/DbcAndTesting.html)
- [Design by contract for Ruby](http://split-s.blogspot.com/2006/02/design-by-contract-for-ruby.html)
- [Contracts in Racket (A Scheme Descendent)](http://pre.plt-scheme.org/docs/html/guide/contracts.html)
- [Contract Soundness for Object-Oriented Languages](http://www.ccs.neu.edu/scheme/pubs/oopsla01-ff.pdf) by Robert Bruce Findler and Matthias Felleisen
- [A Proof Engine for Eiffel](http://tecomp.sourceforge.net/index.php?file=doc/papers/proof/engine)
- *How to Deign Programs* by Matthias Felleisen, Robert Bruce Findler, Matthew Flatt, and Shriram Krishnamurthi [here](http://www.htdp.org/2003-09-26/Book/)


Emacs
-----

Add the following to your .emacs file for better Trammel formatting:

```lisp
    (eval-after-load 'clojure-mode
      '(define-clojure-indent
         (contract 'defun)
         (defconstrainedfn 'defun)
         (defcontract 'defun)
         (provide 'defun)))
```

Example REPL Session
--------------------

Type the following into a REPL session to see how Trammel might be used.

```clojure

    (defconstrainedtype Bar 
      [a b] 
      [(every? pos? [a b])])
    
    (Bar? (->Bar 1 2))
    
    (defn sqr [n] (* n n))
    
    (provide-contracts
      [sqr "the constraining of sqr" 
        [n] [number? (not= 0 n) => pos? number?]])
    
    (sqr 0)

    (positive-nums -1)

    (type (->Bar))
    
    (.a (->Bar  42 77))
    (.b (->Bar  42 77))
    (.a (->Bar -42 77))
    (.b (->Bar  42 -77))

    (defconstrainedfn sqrt
      [x] [(>= x 0) => (>= % 0)]
      (Math/sqrt x))
    
    (defn- bigger-than-zero? [n] (>= n 0))
    
    (defconstrainedfn sqrt
      [x] [bigger-than-zero? => bigger-than-zero?]
      (Math/sqrt x))
    
    (sqrt 10)
    (sqrt -19)
    
    (defconstrainedfn sqrt
      [x] [bigger-than-zero? => bigger-than-zero? (<= (Math/abs (- x (* % %))) 0.01)]
      (Math/sqrt x))
    
    (* (sqrt 30) (sqrt 30))
	
    (def ag (constrained-agent 0
             "only numbers allowed"
             [number?]))
    
    (send ag str)
    
    @ag
    
    (agent-error ag)
    
    (def r (constrained-ref 0
             "only numbers allowed"
             [number?]))
    
    (dosync (alter r inc))
    
    (dosync (alter r str))
    
    (def a (constrained-atom 0
             "only numbers allowed"
             [number?]))
    
    @a
    
    (swap! a inc)
    
    (swap! a str)
    (compare-and-set! a 0 "a")
    
    (defconstrainedvar ^:dynamic foo 0
      "only numbers allowed in Var foo"
      [number?])
    
    (binding [foo :a] [foo])

```
