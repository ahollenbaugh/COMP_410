# COMP 410 Lecture Notes
## Monday, August 28, 2023
- **Abstract syntax tree**
- **Boolean satisfiability (SAT)**
- **Semantic tableau**
- Main question: are there any satisfying solutions to this formula?
- In other words, what are all the possible values of the variable(s) that make the formula true?
- Examples
    - $X \wedge Y$
    - $A \vee B$
    - $\neg X \wedge A$
    - $(A \wedge B) \vee (\neg A \wedge B)$
        - One particular satisfiable solution is A = T, B = T
        - But it’s not the only one!
        - Another: A = F, B = T
    - $X \wedge \neg X$
        - If the # of solutions > 0 then it’s satisfiable
        - This one is unsatisfiable since there are zero solutions
- You can represent circuits with boolean formulae
    - Here's a simple pseudo-boolean formula: $circuit \wedge BadThing$
    - Translation: “Does this circuit result in some bad thing you don’t want to happen?”
    - If no (unsatisfiable), then your circuit is correct! :sunglasses:
    - If yes (satisfiable), then the solution gives the circumstance under which $BadThing$ occurs
    - But what exactly do we mean by "correct"?
        - **Verification** vs. **testing**
            - Testing
                - Prove system incorrect
                - In other words, prove a bug exists
                - Fail test &rarr; bug exists
                - Proof = the inputs you used
            - Verification
                - Prove system correct
                - This is almost never done, it’s too hard! So we do testing
                - Very expensive to make processors, so you really want that stuff to be correct
                - Intel does this ever since floating point wasn’t working 20 years ago, recall, billion dollar loss, “we don’t have _bugs_, we have :sparkles:_specification updates_:sparkles:"
- NP-complete problem
- Worst case scenario these break down to $O(2^n)$
- Different solvers/algos for this
- We’ll look at semantic tableau
    - $A \wedge B \wedge \neg C \wedge \neg D \wedge E \wedge \neg F \wedge \neg G \wedge H$
        - $O(2^8)$
        - Need T, T, F, F, T, F, F, T in order for the whole thing to be T
        - One variable at a time &rarr; $O(n)$, easy with all “and”s, in this case
        - “Or” makes shit harder, have to explore, try multiple situations, you end up with exponential things
            - $(A \wedge B) \vee (\neg A \wedge B)$
    - $X \wedge \neg X$
        - $X$ has to be true, but wait, $X$ has to be false, that’s a conflict -> unsatisfiable
    - $(X \wedge \neg X) \vee Y$
        - From left to right, finds conflict, gets mad, but wait, there’s or! $Y$ has to be true
    - That’s basically how Prolog and semantic tableau work
    - You’ll implement this algo in the first assignment!

## Wednesday, August 30, 2023

- **abstract syntax trees**
    - data structure that represents syntax of some particular input
    - used for compilers that need to reason about program's syntax
    - can use for these formulae
        - we're gonna write a solver for these using ASTs
    - variables and literals are leaves
    - operators are internal nodes
    - these aren't binary trees
- kinds of expressions that can be represented:
    - $X \wedge Y$
    - $2 + 8$
    - $7 * 3$
    - if/else statements
- precedence
    - $1 + 2 * 3$
        - $2 * 3$ has to happen first, deeper in tree
    - implicit parentheses
        - not in tree but they can help determine precedence
            - javascript interpreter exception, paren in tree
    - with AST, all precedence issues will go away
    - `x = 1 + 2 * 3;`
        - `*`, then `+`, then `=`, with implied paren
- parser - function that spits out AST, part of compiler
- you're not writing a parser, you'll be given an AST directly for you to process
- handout, not graded
- negation has one child
- python program
    - one class per kind of AST node
    - `Number(5)` creates AST that only has the number 5
    - `self`: instance of a class, not implicit like `this`
    - every method takes `self` as a param
    - `__str__` lets you return string representation of an object instead of unhelpful "Number object at memory location"
    - `Binop` is an operator that takes two args
    - `Plus(Binop)` extends `Binop`
    - `Plus(Number(1)), Multiply(Number(2), Number(7))`
    - `eval_expr(e)` evals AST
    - `isinstance` checks if instance of a given class, like `Number`
        - depending on the class, either return stored value, or perform given operation
        - eval recursively, on left and right

## Monday, September 25, 2023

- (I was absent last Monday, and I took notes on my iPad Wednesday.)
- unification
    - another core prolog feature
    - equality constraints, how they're handled in the engine
    - examples: 
        - 1 = 1. &rarr; true. 
        - 1 = 2. &rarr; false.
        - X = 1. &rarr; X = 1.
        - 1 = X. &rarr; 1 = X.
    - variables start in an uninstantiated way (no known value)
    - might still have constraint
    - example: X = Y. 
        - both uninstantiated, but still have constraint that X must equal Y
        - it's not clear what X and Y need to be, but it's clear that they need to be equal
    - unification of uninstantiated variables with values forces instantiation
        - (X = 1 forces a value into X, and X will from then on be 1)
- equivalence classes
    - example: {x, y, 1} 
        - set where elements are equivalent to each other
    - effectively what's happening in prolog
    - {x, y, 1} : X = Y, Y = 1
    - start with {x}, {y}, and {1}, perform set union &rarr; end up with {x, y} {1}
    - then, take union of the set containing y with the set containing 1 to get {x, y, 1}
    - not quite a union that happens, there's a check
        - suppose you have X = Y, X = 1, Y = 2
        - {x, y, 1} {2} conflict
        - how can X = Y when each is assigned two different values? 1 != 2
        - treat concrete value as representative of that set (so 1 for first set and 2 for second), then no set unions are necessary, it's just an equality check!
        - upon failure, backtrack to last choice point (another world to explore) - if none left, return false.
        - engine will undo everything since that last point in memory
    - this is effectively what's happening, but the algorithm is really slow and inefficient; prolog uses different method of representation
    - every uninstantiated variable will live somewhere in memory
    - _928 not a memory address, but you could think of it that way
    - leftover value basically
    - X = _2806, Y = _2810
    - after unification: ```X = Y, writeln(X), writeln(Y)``` &rarr; ```_4616``` (mem address of set rep of that variable)
    - set rep not based on position/order of writeln or anything, has to do with data structures
    - represented more as a graph, starting with three nodes in a graph, then unification creates edges (x -> Y) (1)
    - then X -> Y -> 1
    - set rep is the one at the end of the path, if there is a concrete value (can't connect edge from one concrete to another, like 1 -> 2, since remember they're not equivalent) - essentially how this looks in memory
- types of data: uninstantiated variables, integers, atoms
- ints only equal same ints (1 = 1, 1 != 2)
- atoms = same string equals itself ("foo" = "foo", "foo" != "bar")
- can't compare atom to int
- last data type = structure
- handout time :)
- tuple = something that contains other things
- n-ary tuple
- 2-tuples, 3-tuples, 4-tuples, etc. = how many things are inside the tuple
- python -> ```("foo", "bar")```
- elements can be different types (python dynamically typed anyway)
    - ```('foo', 'bar', 1, 2, True, False)```
    - not a set
- sort of related to lists, but these are heterogeneous, every element can have a different type, and tuples have a fixed size, effectively at compile-time, can't dynamically change size
- prolog not dynamically typed
    - if you try to unify atom with integer, they're different types, and you get a failure, not error
- structures are basically tuples with names
- example: ```X = foo(1, 2).```
- ```foo(1, 2).``` &rarr; no assignment means you're trying to call a method called foo with args 1 and 2
- ```=(X, foo(1, 2)).``` &rarr; ```X = foo(1, 2).```
- ```X = call(X), call(X).```
    - cyclic term
    - "stack limit 1 GB exceeded" (stack overflow)
    - prolog lets you make cyclic terms
    - ```unify_with_occurs_check(X, call(X))``` &rarr; occurs check to see if a term contains another term, will do unification if term doesn't contain itself
- atoms can be viewed as a 0-ary tuple, they're just names, and no params
- structures can contain other data, and that other data can itself be a structure
- structures can be partially instantiated
    - ```X = foo(Y, 1)```
1. check structure names and arity - foo(1) = bar(1) or foo(1) = foo(1, 2)
2. parameter-by-parameter, unify - 
foo(X, 5) = 
foo(7, Y)

(We'll finish the handout on Wednesday)


## Wednesday, September 27, 2023

- conjunction works left to right
- so ```X > 6, X = 7``` will result in an error
- use ```X = 7, X > 6``` instead
- ```use_module(library(clpfd))```
- constraint logic programming over finite domains
- ```X #> 7```
- in library
- adds additional constraints
- "finite domains" refer to arithmetic specifically
- libraries for set theory as well
- warning: pino arithmetic? theoretical limitations
- undecidable, infinite loops can occur
- ```X is 1 + 2.```
- ```X = 1 + 2.``` &rarr; ```X = 1+2.```
- infix operators are parsed as structures
- ```Temp = (X is 5 * 2), call(Temp).```
- represents using structure, then evaluates using call
- arithmetic and recursion
- ```trace(name_of_procedure).```
- ```name_of_procedure(0, N).```
- good for debugging
- will show params
- can run as its own separate clause with single-step debugging by running ```trace, fib(0, N).```
- asking for more solutions will ```Redo```
- can get one possible solution and then "false" if you try to get more solutions. This is okay! Doesn't mean it failed, just means there are no more solutions.
- ```!. % cut``` - DO NOT USE! dynamically trims tree of choices. it will break shit.

### Monday, October 2, 2023

- recap
- recursion in prolog
- fibonacci example
- relations, not functions
- base cases = facts usually
- handout
- inherently recursive data structures normally
- numbers don't work that way
- calculating and recursing on a number are not naturally recursive
- that's why fibonacci is a terrible example
- List
    - Cons
    - Nil
- list ::= nil | Cons(elem, list) <- base case vs recursive case
- ```[1, 2, 3]```
- ```X = cons(1, cons(2, cons(3, nil))).```
- prolog has built-in support for lists: ```X = [1, 2, 3].```
- ```X = .(1, .(2, .(3, []))). % . = structure, [] = atom```
- will output nice list (syntactic sugar for the above)
- ```--traditional```
- ```[1, 2, 3] = .(Head, Tail). % will unify head with 1 and tail with rest of list```
- dot is uncommon, and you're not really supposed to do this
- ```[1, 2, 3] = [Head | Tail]. % shorthand, more common```
- this is like 90% of what you need to know about lists in Pl
- can use same syntax to destruct and construct a list
- ```Head = 6, Tail = [1, 3], List = [Head | Tail].```
- ```myAppend``` (check if file on website)
- can run this in reverse, too
- give result list, it will output every pair of lists
- this is just kind of a preview of what we'll cover wednesday

### Wednesday, October 4, 2023

- Exam next Wednesday
- Practice exam will be posted tomorrow
- Monday = review day
- Pencil 'n' paper
- tail recursion optimization
- no mutable var -> no loop -> forced recursion
- you only use local variables if you do something AFTER recursive call
- [Head|Tail] separates a list into head and tail

### Wednesday, October 18, 2023

- List = [1, 2, 3]
- List = .(1, .(2, .(3, [])))
    - .() is structure cons
    - [] is atom nil
- 1.pl --traditional
    - ?
- functor(foo(a, b), Name, Arity).
- functor([], Name, Arity).
- trace(sum) % excellent debugging feature

### Wednesday, October 25, 2023

- Last time: turn non-recur procedures into recursive ones
- handout - we did #1 and #3
- deconstruct list first then add, inside out