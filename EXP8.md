_______FORWARD CHAINING_____________________

% Define some facts

bird(sparrow).

bird(eagle).

mammal(cat).

mammal(dog).

% Define rules

can_fly(X) :- bird(X).

has_fur(X) :- mammal(X).

% Query to perform forward chaining

?- can_fly(X).


________BACKWARD CHAINING______________________

% Facts
parent(john, mary).
parent(john, lisa).
parent(mary, ann).
parent(lisa, pat).

% Rules
ancestor(X, Y) :- 
    parent(X, Y).                       % Base case: direct parent

ancestor(X, Y) :- 
    parent(X, Z), 
    ancestor(Z, Y).                     % Recursive case

% -----------------------------
% Example Queries:
% -----------------------------
% ?- ancestor(john, pat).
% ?- ancestor(john, ann).
% ?- ancestor(mary, pat).
% ?- ancestor(X, pat).
% ?- ancestor(john, Y).







As the name implies, forward chaining begins with known facts and moves forward by implementing logical rules to extract more information, and it keeps going until it reaches the objective, whereas backward chaining begins with the goal and moves backward by implementing logical deduction rules to evaluate the information that satisfies the objective.

Backward chaining is a goal-driven inference method, whereas forward chaining is a data-driven inference technique.

Forward chaining is referred to as the down-up method, whereas backward chaining is referred to as the top-down approach.

Backward chaining employs a depth-first search strategy, whereas forward chaining employs a breadth-first search strategy.

Both forward and reverse chaining use the Modus poneus reasoning rule
