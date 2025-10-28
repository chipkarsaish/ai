% Initial and Goal states
initial_state([on(a, table), on(b, table), on(c, a)]).
goal_state([on(a, b), on(b, table), on(c, table)]).

% Check if a block is clear in a given State (no block on top of it)
clear(Block, State) :-
    \+ member(on(_, Block), State).

% Remove a fact from the state list
remove(Fact, State, NewState) :-
    select(Fact, State, NewState).

% Add a fact to the state list if not already present
add(Fact, State, NewState) :-
    \+ member(Fact, State),
    NewState = [Fact | State].
add(Fact, State, State) :-
    member(Fact, State).

% Pickup action: pick up Block from From in State producing NewState
move(State, pickup(Block, From), NewState) :-
    member(on(Block, From), State),
    clear(Block, State),
    remove(on(Block, From), State, TempState),
    add(holding(Block), TempState, NewState).

% Putdown action: put down Block on To in State producing NewState
move(State, putdown(Block, To), NewState) :-
    member(holding(Block), State),
    clear(To, State),
    remove(holding(Block), State, TempState),
    add(on(Block, To), TempState, NewState).

% Depth-first search planner
dfs(State, Goal, _, []) :-
    subset(Goal, State). % Goal satisfied if all goal facts are in current state

dfs(State, Goal, Visited, [Action|Plan]) :-
    move(State, Action, NewState),
    \+ member(NewState, Visited),
    dfs(NewState, Goal, [NewState|Visited], Plan).

% Main planning predicate
plan(Plan) :-
    initial_state(InitialState),
    goal_state(Goal),
    dfs(InitialState, Goal, [InitialState], Plan).
