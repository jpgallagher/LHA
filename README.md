# Steps to follow to verify LHA

Linear Hybrid Automata (LHA) is a variant of Hybrid Automata in which systems with linear dynamics could be specified. Such LHA models could then be subjected to **reachability analysis** as well as **model checking**. 

## LHA Reachability Analysis (Part 1) 
For doing reachability analysis and/or model checking the LHA model, the first step is to compile the LHA into a state transition system representation. This representation is a constraint logic program comprising clauses `rState(Xs):- trans(Xp, Xs), rState(Xp).`

### Compiling LHA into a transition system
In this step we use the LHA compilation script `lhac` by passing it with the `LHA file` to be compiled. This LHA compilation script, first flattens the LHA and secondly specialises that flattened file with the `logen` tool.

#### Usage 
```sh
lhac lhaFileWithNoEvents
```
This input `lhaFile` gets transformed into `lhaFileWithNoEvents.spec`. This lhaFile is an LHA with no events.

#### Note
LHAs could be either with events or without events. If the LHA is with events i.e. `lhaFileWithEvents`. The usage with events is as following:
 
#### Usage 
```sh
lhac -e lhaFileWithEvents
```

## LHA Reachability Analysis (Part 2)
Reachability analysis mandates computing the set of reachable states. From the specialised transition system `*.spec` , we need to compute the reachable state space for that LHA using Convex Hull Analyser `cha.pl` from the CTL directory from within `Ciao prompt`. This is explained below:

1. First, launch ciao from command prompt, as below:
```sh
ciao
```
2. From ciao prompt
a. consult the file `cha.pl` as below:
```sh
Ciao 1.18.0 (2019-04-17 16:20:55 +0200) [LINUXx86_64]
?- use_module(cha).
```

b. then invoke the exported predicate "go/1" by passing into it the specialised `lhaFile.spec` as below:
```sh
?- go('Tests2019/lhaWaterLevelMonitorWithNoEventsNew.spec')
```
This call outputs a file called `versions.out` with reachable regions from every location in the *LHA*. 
In the above, argument 'Tests2019/lhaWaterLevelMonitorWithNoEventsNew.spec', we have the files in the `Tests2019` folder, which is in the cwd of the ciao shell.

## Towards Modelchecking the LHA (Part 1)
For precision in abstract interpretation based model checking, which we shall do later, it is desirable that if the reachable state space regions are overlapping-regions then they need be refined into partition i.e. disjoint regions. This is done by `makeDisjoint.pl`.

### Usage
1. First, launch ciao from command prompt, as below:
```sh
ciao
```
2. From ciao prompt
a. consult the file `makeDisjoint.pl` as below:
```sh
Ciao 1.18.0 (2019-04-17 16:20:55 +0200) [LINUXx86_64]
?- use_module(makeDisjoint).
```

b. then invoke the exported predicate makeDisjoint:main/1 as following:
```sh
Ciao 1.18.0 (2019-04-17 16:20:55 +0200) [LINUXx86_64]
?- makeDisjoint:main(['Tests2019/versions.out']).
```
This generates the disjoint regions in a file `versions.out.disjoint` under the `Tests2019` folder.

## Towards Modelchecking the LHA (Part 2)
In modelchecking, the algorithm requires predecessors of all region, during fixed-point computation routine, to improve the performance, we precompute these predecessors for regions in the form of `*.predExistsTable`. This is the purpose of `predExistsTable.pl` in the CTL folder.
### Usage
1. Consult the `predExistsTable.pl` from ciao prompt: 
```sh
Ciao 1.18.0 (2019-04-17 16:20:55 +0200) [LINUXx86_64]
?- use_module(predExistsTable).
```

2. Invoke the predicate to generate the `*.predExistsTable` as: 
```sh
Ciao 1.18.0 (2019-04-17 16:20:55 +0200) [LINUXx86_64]
?- predExistsTable:go('Tests2019/lhaWaterLevelMonitorWithNoEventsNew.spec', 'Tests2019/versions.out.disjoint').
```

## Towards Modelchecking the LHA (Final Part)
Modelchecking with concrete state space based regions. The modelchecker has to be input with three files:
1. Spec file of LHA  `lhaFile.spec`;
2. The disjoint regions file `versions.out.disjoint`
3. The predExistsTable file `lhaFile.predExistsTable`
4. The negated version of the CTL formula to be modelchecked 
For example, this could be if formula is ag(x=10), then negated form would be neg(ag(x=10)). 
On execution, this modelchecker returns the set of states where this formula holds. If this does not include the initial states region, the formula holds on the system

### Usage
The predicate to be invoked is: testId/5 from amc_all.pl i.e. testId('T1',F1,V1,T1, PhiT1), where the arguments are as following:
1. Argument T1 is a "label representation for the table being used"
2. Argument F1 is the "lha.spec" file
3. Third argument V1 is the regions containing "versions.out" file
4. Fourth argument T1 is the predExistsTable file "lha.predExistsTable" and
5. Fifth argument PhiT1 is the negation of the formula to be model checked.

This testId/5 is defined in the amc_all.pl. 

A simplified access to testId/5 is illustrated in: 1. amcnew_tests_2019.pl and 2. amcnew_tests.pl. This is reproduced below for reference.
````
testExs2019 :-
	F1 = 'Tests2019/lhaWaterLevelMonitorWithNoEventsNew.spec',
	V1 = 'Tests2019/versions.out',
	T1 = 'Tests2019/lhaWaterLevelMonitorWithNoEventsNew.predExistsTable',
	numbervars((_X1,X2,_X3,_X4,_X5),0,_),
	PhiT1 = neg(ef(prop(X2=10))), testId('T1',F1,V1,T1, PhiT1),
	PhiT2 = neg(ag(and(prop(X2>=1),prop(X2 =< 12)))), testId('T2',F1,V1,T1, PhiT2).
````
# References

1. WCLP 

2. LPAR

3. LOPSTR

4. LOGEN 
