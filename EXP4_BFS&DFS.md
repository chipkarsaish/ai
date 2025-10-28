BFS --------------------------------
% Define edges
edge(a, b).
edge(a, c).
edge(b, d).
edge(b, e).
edge(c, f).
edge(c, g).
edge(d, h).
edge(d, i).

% BFS predicate
bfs(Start, Goal) :-
    bfs_queue([[Start]], Goal).

% Goal found, print solution
bfs_queue([[Goal|Path]|_], Goal) :-
    reverse([Goal|Path], Solution),
    write('Solution: '), write(Solution), nl.

% Expand current path, add new paths to the queue
bfs_queue([Path|Paths], Goal) :-
    extend(Path, NewPaths),
    append(Paths, NewPaths, UpdatedPaths),
    bfs_queue(UpdatedPaths, Goal).

% Extend path by exploring neighboring nodes
extend([Node|Path], NewPaths) :-
    findall([NewNode, Node|Path],
            (edge(Node, NewNode), \+ member(NewNode, [Node|Path])),
            NewPaths),
    !.
extend(_, []). % No more extensions

% Handle case where no solution exists
bfs_queue([], _) :-
    write('No solution found'), nl.



DFS-----------------------------------------------------------------------------
% Define edges
edge(a, b).
edge(a, c).
edge(b, d).
edge(b, e).
edge(c, f).

% DFS predicate: finds a path from Start to End
dfs(Start, End, Path) :-
    dfs_helper(Start, End, [], Path).

% Base case: reached the destination
dfs_helper(Node, Node, Visited, [Node|Visited]).

% Recursive case: traverse to adjacent nodes
dfs_helper(Current, End, Visited, Path) :-
    edge(Current, Next),
    \+ member(Next, Visited), % Avoid revisiting nodes
    dfs_helper(Next, End, [Current|Visited], Path).

% Predicate to start DFS and print the result
dfs_start(Start, End) :-
    dfs(Start, End, Path),
    reverse(Path, PathReversed),
    format("DFS path from ~w to ~w: ~w", [Start, End, PathReversed]).

