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


