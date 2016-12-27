# Assignment 2 #
This assignment is to be carried out in groups of two or three students (please record the composition of your group via supplied Google Form before starting the assignment). It consists of three parts and an optional part, described below.

## Submission and marking guidelines ##

1. The only Prolog files to be submitted are `assignment2_12345.pl` and `assignment2_wp_12345.pl` (where `12345` is candidate number of chosen group member).
2. If you aim for a mark of 70% you do not need to submit anything else. We will test your program on a number of random configurations, and you will get 70% if (for each not satisfied requirement we will deduct marks pro-rata):
    1. your program solves most of them,
    2. without running out of fuel,
    3. following more-or-less shortest paths,
    4. on grid sizes that are slightly larger or smaller than 20, and
    5. without relying on the internals of the provided library.
3. Additional marks can be gained by doing the optional part. If you want to claim additional marks, please submit a short report in PDF explaining what you have done to merit them.

## Part 1: Getting around ##
You need to be able to navigate a space populated by obstacles. Every move depletes your energy and so you frequently need to visit a charging station for a top-up. A basic depth-first search strategy is given but you need to improve this to get around efficiently.

Brief instructions:

1. Download the latest coursework file [here](https://github.com/COMS30106/assignment/releases/latest) to get started, unzip it and go to the root directory. **Do not move or rename any files.**
2. To run this assignment execute: `./ailp.pl assignment2 part1`.
2. The file `assignment2_12345.pl`contains a simple backtracking search predicate `solve_task/2`; you need to change this to apply A\* search.
3. Update your candidate number in both filename and `candidate_number/1` predicate inside the file.
4. Assignment 2 library file (`ailp/library/oscar_library.pl`) contains a simple command shell `shell/0` which might make interaction easier (you can invoke it with `?- shell.`), although you don't have to use it. In the lectures I demonstrated the following sequence of commands, which leave you out of energy without a charging station in sight so your only option is to `reset.` the board:

        ? reset.
        ? find(o(1)).
        ! [cost(5),depth(5)]
        ? ask(o(1),'What is the meaning of life, the universe and everything?').
        ! 42
        ? go(p(7,7)).
        ! [cost(9),depth(9)]
        ? energy.
        ! current_energy(66)
        ? position.
        ! current_position(p(7,7))
        ? go(p(10,10)).
        ! This failed.
        ? energy.
        ! current_energy(0)
        ? position.
        ! current_position(p(20,8))
        ? call(map_adjacent(p(20,8),P,O)).
        ! map_adjacent(p(20,8),p(20,9),c(3))
        ? topup(c(3)).
        ? energy.
        ! current_energy(80)
        ? go(p(10,10)).
        ! This failed.
        ? energy.
        ! current_energy(0)

5. The file `ailp/library/oscar_library.pl` defines the predicates you can use (see the module declaration at the top). Don't change this file and don't change any of the `internal_*` predicates.
6. The given grid world is 20 by 20, has four charging stations, one oracle and about 80 obstacles. Locations and numbers may change when testing your code. Adapt the predicate `init_things/0` if you want to generate your own random configurations of obstacles for testing purposes.

**Hints:**

1. The first step is to change backtracking search into agenda-based search. Where `solve_task_bt` has an input variable `Current` you need to change that to an agenda (a list).
2. The format of `Current` in the given code is `[c(Cost,Pos)|RPath]` where `Pos` is the current position, `RPath` is the reverse path up to and including the current position, and `Cost` is the cost (i.e., number of steps) of reaching the current position.
3. The agenda needs to hold a list of such structures. Children to be added to the agenda can be generated by an appropriate `setof/3` call involving `search/4`.
4. For A\* search `Cost` would be the F-value of a node, where F=G+H, G is the distance travelled, H is estimated by the Manhattan distance to the goal.
5. Note that putting `Cost` at the front of the `[c(Cost,Pos)|RPath]` structure ensures that `setof/3` generates children in the order of increasing F-value, which is convenient because it simplifies adding the children to the agenda.
6. I find it generally easiest to maintain both F- and G-values for each node on the agenda, which would lead to the structure `[c(F,G,Pos)|RPath]`. You are free to modify these functors, as long as you do it consistently; as Prolog is an untyped language these functors are "declared" by usage.
7. For tasks of the form `find(o(1))` you don't know the location of the goal and so can't estimate H; in this case your search strategy should be breadth-first search so that you still find an optimal path. This could be achieved by setting H=0 for all children of the current position.

## Part 2: Identity crisis ##
You know you are a famous actor or actress at the forthcoming Academy Awards ceremony, but you have forgotten who you are. Oracles can give you clues about your identity. `wp_library.pl` file located in `ailp/library` directory defines a Prolog interface to Wikipedia. For example, the following query lists all links found on a particular Wikipedia page:

        ?-wp('Peter Flach',WT),wt_link(WT,L),writeln(L),fail.
          Sneek
          data mining
          machine learning
          University of Bristol
          Universiteit Twente
          PhD
          Universiteit van Tilburg
          Bristol University
          Wikipedia:Persondata
          Category:1961 births
          Category:Living people
          Category:Artificial intelligence researchers
          Category:Dutch computer scientists
          Category:British computer scientists
          Category:Machine learning researchers
          Category:People from Sneek
          Category:University of Twente alumni
          Category:Tilburg University alumni
          Category:Academics of the University of Bristol

(Note: this is a particular Prolog technique called a **failure-driven loop**.)

To run this assignment execute: `./ailp.pl assignment2 part2`. Every time `wp_library.pl` is loaded it will assert a random secret identity from among those enumerated by `actor/1`. You can find out information about this secret identity by means of `?-agent_ask_oracle(oscar,o(1),link,L).`, which will return a random link `L` from the secret identity's Wikipedia page (actually, from a specific subset enumerated by `link/1`).

Your mission, should you choose to accept it, involves the implementation of a predicate `find_identity/1` (located in `assignment2_wp_12345.pl` file) which repeatedly calls `agent_ask_oracle(oscar,o(1),link,L)` and deduces the secret identity by a process of elimination. You can test your code by means of the query `?-test`, which iterates through all possible secret identities.

**Hint:** with the given code to query Wikipedia this is easier than it may sound: my solution has less than 20 lines of Prolog code.

## Part 3: Identity crisis, continued ##
Now you need to adapt `find_identity/1` to work in the grid world. Running the assignment with `./ailp.pl assignment2 part3` will create random 20 by 20 grid worlds with 10 oracles, 2 charging stations and up to 100 obstacles. You can get a random link from each oracle that will help you to deduce the secret identity as before. The catch is that each oracle can be queried only once. Furthermore, querying the oracle costs 10 units of energy, so you need to keep your energy levels topped up (at the start and after each top-up your energy level is 100). Here is a QuickTime video to demonstrate the main idea: [`oscar480.mov`](https://www.cs.bris.ac.uk/Teaching/Resources/COMS30106/labs/assignment2/oscar480.mov).

**Hints:**
1. Run the assignment with `./ailp.pl assignment2 part3`.
2. Modify `find_identity/1` to navigate around the grid world, find oracles and ask for random links.
3. It is a good idea for your agent to maintain an internal memory of where it has seen oracles and charging stations, so that it can easily navigate to the nearest charging station and (unvisited) oracles. In my example this internal memory is reset using `init_oscar_memory/0`.
4. The only change to the `oscar_library.pl` API is an added predicate `agent_check_oracle/2` which succeeds if your agent (`oscar`) has already queried this oracle.
5. The worlds are entirely randomly generated, which means that some worlds are unsolvable (e.g., you may not be able to leave the starting position). Also, the oracles are independent and are likely to give you a link you already knew, so sometimes you don't know your identity even after visiting all oracles.

## Part 4: Optional ##
You will need to navigate a dynamic world, as some of the obstacles, oracles and charging stations may be moving around, and the world may contain additional agents. It is likely that your previous A\* search will fail whenever the agent finds its planned path blocked by a moved object or another agent. So the minimum I'm looking for here is robustness in your search and the ability to re-plan when necessary. Being able to work with multiple agents is a bonus.

Brief instructions:
1. To run the assignment execute: `./ailp.pl assignment2 part4`.
2. One change is that the code can accommodate multiple agents. Communication with the world model is therefore now via HTTP. See *part 4* section of the `ailp/library/oscar_library.pl` file, which explains the API and contains *shell* with some added commands.
3. Another change is that the objects in the world can move around in a random fashion. The parameters controlling this can be changed in the file `ailp/library/game_predicates.pl`.
4. `assignment2_12345.pl` contains the same simple backtracking search algorithm given before.
5. The main steps to get your own code working with this involve changing calls like `agent_current_position(Agent,Pos)` to `query_world(agent_current_position,[Agent,Pos])`.