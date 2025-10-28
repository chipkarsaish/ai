% --- Move rules ---
move(S, Snew):- right(S, Snew).
move(S, Snew):- left(S, Snew).
move(S, Snew):- down(S, Snew).
move(S, Snew):- up(S, Snew).

right([R1,R2,R3,R4,R5,R6,R7,R8,R9],Snew):- R3>0, R6>0, R9>0, blank_right([R1,R2,R3,R4,R5,R6,R7,R8,R9],Snew).
blank_right([R1,R2,R3,R4,R5,R6,R7,R8,R9],S):-
    nth0(N,[R1,R2,R3,R4,R5,R6,R7,R8,R9],0),
    Z is N+1,
    nth0(Z,[R1,R2,R3,R4,R5,R6,R7,R8,R9],R),
    substitute(R,[R1,R2,R3,R4,R5,R6,R7,R8,R9],10,Q),
    substitute(0,Q,R,V),
    substitute(10,V,0,S).

left([R1,R2,R3,R4,R5,R6,R7,R8,R9],Snew):- R1>0, R4>0, R7>0, blank_left([R1,R2,R3,R4,R5,R6,R7,R8,R9],Snew).
blank_left([R1,R2,R3,R4,R5,R6,R7,R8,R9],S):-
    nth0(N,[R1,R2,R3,R4,R5,R6,R7,R8,R9],0),
    Z is N-1,
    nth0(Z,[R1,R2,R3,R4,R5,R6,R7,R8,R9],R),
    substitute(R,[R1,R2,R3,R4,R5,R6,R7,R8,R9],10,Q),
    substitute(0,Q,R,V),
    substitute(10,V,0,S).

down([R1,R2,R3,R4,R5,R6,R7,R8,R9],Snew):- R7>0, R8>0, R9>0, blank_down([R1,R2,R3,R4,R5,R6,R7,R8,R9],Snew).
blank_down([R1,R2,R3,R4,R5,R6,R7,R8,R9],S):-
    nth0(N,[R1,R2,R3,R4,R5,R6,R7,R8,R9],0),
    Z is N+3,
    nth0(Z,[R1,R2,R3,R4,R5,R6,R7,R8,R9],R),
    substitute(R,[R1,R2,R3,R4,R5,R6,R7,R8,R9],10,Q),
    substitute(0,Q,R,V),
    substitute(10,V,0,S).

up([R1,R2,R3,R4,R5,R6,R7,R8,R9],Snew):- R1>0, R2>0, R3>0, blank_up([R1,R2,R3,R4,R5,R6,R7,R8,R9],Snew).
blank_up([R1,R2,R3,R4,R5,R6,R7,R8,R9],S):-
    nth0(N,[R1,R2,R3,R4,R5,R6,R7,R8,R9],0),
    Z is N-3,
    nth0(Z,[R1,R2,R3,R4,R5,R6,R7,R8,R9],R),
    substitute(R,[R1,R2,R3,R4,R5,R6,R7,R8,R9],10,Q),
    substitute(0,Q,R,V),
    substitute(10,V,0,S).

% --- Helper predicate for swapping ---
substitute(_, [], _, []):- !.
substitute(X, [X|T], Y, [Y|T1]):- substitute(X, T, Y, T1), !.
substitute(X, [Y|T], Y, [X|T1]):- substitute(X, T, Y, T1), !.
substitute(X, [H|T], Y, [H|T1]):- substitute(X, T, Y, T1).

% --- A* Search Start ---
go(Start,Goal):-
    getHeuristic(Start, H, Goal),
    path([[Start,null, 0, H, H]],[],Goal).

% --- A* Main Logic ---
path([], _, _):- write('No solution'),nl,!.
path(Open, Closed, Goal):-
    getBestChild(Open, [Goal, Parent, PC, H, TC], _),
    write('A solution is found'),  nl ,
    printsolution([Goal,Parent, PC, H, TC], Closed),!.

path(Open, Closed, Goal):-
    getBestChild(Open, [State, Parent, PC, H, TC], RestOfOpen),
    getchildren(State, Open, Closed, Children, PC, Goal),
    addListToOpen(Children , RestOfOpen, NewOpen),
    path(NewOpen, [[State, Parent, PC, H, TC] | Closed], Goal).

% --- Get valid children states ---
getchildren(State, Open ,Closed , Children, PC, Goal):-
    bagof(X, moves(State, Open, Closed, X, PC, Goal), Children).
getchildren(_,_,_,[],_,_).

% --- Add children states to open list ---
addListToOpen(Children, [], Children).
addListToOpen(Children, [H|Open], [H|NewOpen]):-
    addListToOpen(Children, Open, NewOpen).

% --- Select best state from open list ---
getBestChild([Child], Child, []).
getBestChild(Open, Best, RestOpen):-
    getBestChild1(Open, Best),
    removeFromList(Best, Open, RestOpen).

getBestChild1([State], State).
getBestChild1([State|Rest], Best):-
    getBestChild1(Rest, Temp),
    getBest(State, Temp, Best).

getBest([State, Parent, PC, H, TC], [_, _, _, _, TC1], [State, Parent, PC, H, TC]):-
    TC < TC1, !.

getBest([_, _, _, _, _], [State1, Parent1, PC1, H1, TC1], [State1, Parent1, PC1, H1, TC1]).

% --- Remove an item from a list ---
removeFromList(_, [], []).
removeFromList(H, [H|T], V):-!, removeFromList(H, T, V).
removeFromList(H, [H1|T], [H1|T1]):- removeFromList(H, T, T1).

% --- Generate next moves ---
moves(State, Open, Closed, [Next,State, NPC, H, TC], PC, Goal):-
    move(State,Next),
    \+ member([Next, _, _, _, _],Open),
    \+ member([Next, _, _, _, _],Closed),
    NPC is PC + 1,
    getHeuristic(Next, H, Goal),
    TC is NPC + H.

% --- Heuristic: count misplaced tiles ---
getHeuristic([], 0, []) :- !.
getHeuristic([H|T1], V, [H|T2]) :- !, getHeuristic(T1, V, T2).
getHeuristic([_|T1], H, [_|T2]) :-
    getHeuristic(T1, TH, T2),
    H is TH + 1.

% --- Print solution path ---
printsolution([State, null, PC, H, TC],_):-
    write(State), write(' PC: '), write(PC), write(' H:'), write(H), write(' TC: '), write(TC), nl.

printsolution([State, Parent, PC, H, TC], Closed):-
    member([Parent, GrandParent, PC1, H1, TC1], Closed),
    printsolution([Parent, GrandParent, PC1, H1, TC1], Closed),
    write(Parent), write(State), write(' !!PC: '), write(PC), write(' H:'), write(H), write(' TC: '), write(TC), nl.










    -----------------command---------
    ?- go([1,2,3,4,5,6,7,8,0],[1,2,3,4,5,6,7,8,0]).

