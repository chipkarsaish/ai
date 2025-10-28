:- dynamic board/1.

init:- 
   retractall(board(_)),
   assert(board([_Z1,_Z2,_Z3,_Z4,_Z5,_Z6,_Z7,_Z8,_Z9])).
:- init.

mark(Player, [X|_],1,1) :- var(X), X=Player.
mark(Player, [,X|],2,1) :- var(X), X=Player.
mark(Player, [,,X|_],3,1) :- var(X), X=Player.
mark(Player, [,,,X|],1,2) :- var(X), X=Player.
mark(Player, [,,,,X|_],2,2) :- var(X), X=Player.
mark(Player, [,,,,,X|],3,2) :- var(X), X=Player.
mark(Player, [,,,,,,X|_],1,3) :- var(X), X=Player.
mark(Player, [,,,,,,,X|],2,3) :- var(X), X=Player.
mark(Player, [,,,,,,,,X|_],3,3) :- var(X), X=Player.

move(P,(1,1),[X1|R],[P|R]) :- var(X1).
move(P,(2,1),[X1,X2|R],[X1,P|R]) :- var(X2).
move(P,(3,1),[X1,X2,X3|R],[X1,X2,P|R]) :- var(X3).
move(P,(1,2),[X1,X2,X3,X4|R],[X1,X2,X3,P|R]) :- var(X4).
move(P,(2,2),[X1,X2,X3,X4,X5|R],[X1,X2,X3,X4,P|R]) :- var(X5).
move(P,(3,2),[X1,X2,X3,X4,X5,X6|R],[X1,X2,X3,X4,X5,P|R]) :- var(X6).
move(P,(1,3),[X1,X2,X3,X4,X5,X6,X7|R],[X1,X2,X3,X4,X5,X6,P|R]) :- var(X7).
move(P,(2,3),[X1,X2,X3,X4,X5,X6,X7,X8|R],[X1,X2,X3,X4,X5,X6,X7,P|R]) :- var(X8).
move(P,(3,3),[X1,X2,X3,X4,X5,X6,X7,X8,X9|R],[X1,X2,X3,X4,X5,X6,X7,X8,P|R]) :- var(X9).

record(Player,X,Y) :- 
   retract(board(B)), 
   mark(Player,B,X,Y),
   assert(board(B)).

win([Z1,Z2,Z3|_],P) :- Z1==P, Z2==P, Z3==P.
win([,,,Z1,Z2,Z3|],P) :- Z1==P, Z2==P, Z3==P.
win([,,,,,,Z1,Z2,Z3],P) :- Z1==P, Z2==P, Z3==P.
win([Z1,,,Z2,,,Z3,,],P) :- Z1==P, Z2==P, Z3==P.
win([,Z1,,,Z2,,,Z3,],P) :- Z1==P, Z2==P, Z3==P.
win([,,Z1,,,Z2,,,Z3],P) :- Z1==P, Z2==P, Z3==P.
win([Z1,,,,Z2,,,,Z3],P) :- Z1==P, Z2==P, Z3==P.
win([,,Z1,,Z2,,Z3,,],P) :- Z1==P, Z2==P, Z3==P.

open([Z1,Z2,Z3|_],Player) :- (var(Z1); Z1==Player),(var(Z2); Z2==Player),(var(Z3); Z3==Player).
open([,,,Z1,Z2,Z3|],Player) :- (var(Z1); Z1==Player),(var(Z2); Z2==Player),(var(Z3); Z3==Player).
open([,,,,,,Z1,Z2,Z3],Player) :- (var(Z1); Z1==Player),(var(Z2); Z2==Player),(var(Z3); Z3==Player).
open([Z1,,,Z2,,,Z3,,],Player) :- (var(Z1); Z1==Player),(var(Z2); Z2==Player),(var(Z3); Z3==Player).
open([,Z1,,,Z2,,,Z3,],Player) :- (var(Z1); Z1==Player),(var(Z2); Z2==Player),(var(Z3); Z3==Player).
open([,,Z1,,,Z2,,,Z3],Player) :- (var(Z1); Z1==Player),(var(Z2); Z2==Player),(var(Z3); Z3==Player).
open([Z1,,,,Z2,,,,Z3],Player) :- (var(Z1); Z1==Player),(var(Z2); Z2==Player),(var(Z3); Z3==Player).
open([,,Z1,,Z2,,Z3,,],Player) :- (var(Z1); Z1==Player),(var(Z2); Z2==Player),(var(Z3); Z3==Player).

value(Board,100) :- win(Board,o), !.
value(Board,-100) :- win(Board,x), !.
value(Board,E) :- 
   findall(o,open(Board,o),MAX), 
   length(MAX,Emax), 
   findall(x,open(Board,x),MIN), 
   length(MIN,Emin), 
   E is Emax - Emin.

:- assert(lookahead(2)).
:- dynamic spy/0.
:- assert(spy).

search(Position,Depth,(Move,Value)) :- 
   alpha_beta(o,Depth,Position,-100,100,Move,Value).

alpha_beta(Player,0,Position,_Alpha,_Beta,_NoMove,Value) :- 
   value(Position,Value),
   spy(Player,Position,Value).

alpha_beta(Player,D,Position,Alpha,Beta,Move,Value) :- 
   D > 0, 
   findall((X,Y),mark(Player,Position,X,Y),Moves), 
   Alpha1 is -Beta, 
   Beta1 is -Alpha,
   D1 is D-1, 
   evaluate_and_choose(Player,Moves,Position,D1,Alpha1,Beta1,nil,(Move,Value)).

evaluate_and_choose(Player,[Move|Moves],Position,D,Alpha,Beta,Record,BestMove) :-
   move(Player,Move,Position,Position1), 
   other_player(Player,OtherPlayer),
   alpha_beta(OtherPlayer,D,Position1,Alpha,Beta,_OtherMove,Value),
   Value1 is -Value,
   cutoff(Player,Move,Value1,D,Alpha,Beta,Moves,Position,Record,BestMove).
evaluate_and_choose(_Player,[],_Position,_D,Alpha,_Beta,Move,(Move,Alpha)).

cutoff(_Player,Move,Value,_D,_Alpha,Beta,_Moves,_Position,_Record,(Move,Value)) :- 
   Value >= Beta, !.
cutoff(Player,Move,Value,D,Alpha,Beta,Moves,Position,_Record,BestMove) :- 
   Alpha < Value, Value < Beta, !, 
   evaluate_and_choose(Player,Moves,Position,D,Value,Beta,Move,BestMove).
cutoff(Player,_Move,Value,D,Alpha,Beta,Moves,Position,Record,BestMove) :- 
   Value =< Alpha, !, 
   evaluate_and_choose(Player,Moves,Position,D,Alpha,Beta,Record,BestMove).

other_player(o,x).
other_player(x,o).

spy(Player,Position,Value) :- 
   spy, !,
   write(Player),
   write(' '),
   write(Position),
   write(' '),
   writeln(Value).
spy(,,_).

h(X,Y) :- 
   record(x,X,Y), 
   showBoard.

c :- 
   board(B), 
   alpha_beta(o,2,B,-200,200,(X,Y),_Value), 
   record(o,X,Y),
   showBoard.

showBoard :- 
   board([Z1,Z2,Z3,Z4,Z5,Z6,Z7,Z8,Z9]), 
   write('    '),mark(Z1),write(' '),mark(Z2),write(' '),mark(Z3),nl,
   write('    '),mark(Z4),write(' '),mark(Z5),write(' '),mark(Z6),nl,
   write('    '),mark(Z7),write(' '),mark(Z8),write(' '),mark(Z9),nl.
s :- showBoard.

mark(X) :- var(X), write('#').
mark(X) :- \+var(X), write(X).




-----commands-----
?- s.
?- h(2,2).
?- c.
?- h(3,3).
?- c.

