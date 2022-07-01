
This is a readme file for the toy example on SMT solver, Z3. 

## Software Used
1. [Z3-Sovler](https://github.com/Z3Prover/z3)

## Setup
1. Install the Z3 solver with python interface using pip: `pip install z3-solver>=4.8.14.0`

## Files
1. `Toy_Example_SMT.ipynb :` It contains the script for toy example (illustration of our problem) in jupyter notebook platform.
2. `SMT_Problem_Description.pdf`: It contains the Description of the problem that we are facing.

## Functions Defined
1. `eqn1(S)`: This function will take an array `S` as an input and return either a value or an equation depending upon whether the entries of `S` are `0/1` or some variables.
2. `eqn2(A)`: This function will take an array `A` as an input and return the value of an equation when applied on `A`.
3. `update(S)`: This function will update the array `S` with new values or variables depending upon whether the entries of `S` are `0/1` or variables.
4. `toy(...)`: This function contains the SMT modelling for a random array. 

## Description
First we define a random array, `array_org` of size `array_size` with entries as `0/1`. Thereafter, we divide/partition the array `S` into consecutive parts, each of size `prt_len`. After that we collect the value of equation `eqn1` from the array `S` and the value of equation `eqn2` in from each part of array `S`.
Next, we update the array `S` and collect the value of `eqn1` and `eqn2` from the updated array `S` in the same way as previously. We continue this up to `N` times.
All values of `eqn1` are stored in the array `Z` and the values of `eqn2` are stored in the array `prt_val`.

### Case 1: error_flag = 1
In this case, we will pick a number randomly from `Error_range = [-error, -error + 1, ..., error]` and add it to the entries of the array `prt_val`. For each entries, different number will be picked from `Error_range`.
`prt_val` is updated with the new erroneous values. 

### Case 2: error_flag = 0
In this case, we will inject errors only to the last 30% positions of the array `prt_val`, errors are picked randomly from `Error_range`.

In both of the above cases, positions for error injection are unknown during SMT modelling.

### SMT Modelling
Now, we want to recover `array_org` with the information of `Z`, `prt_val` and the above defined functions. Therefore, we defined an array `arr_var_org` consisting of `BitVec` variables, each of 1-bit, to represent the target array `array_org`. Assign `S = arr_var_org[:]`. Thereafter we form `eqn1` on the defined variables, `S` and equate it to the known values, i.e., `Z`. For equation 2 we will proceed as follows:

For `Case 1`: We will form the costraints as follows: First form the `eqn2` from array `S`, then the constraints are
```
prt_val[i,j] - error <= eqn2 [i,j] <= prt_val[i,j] + error
```
, as we know that all errors are in the range `[-error, -error + 1, ..., error]`.

For `Case 2`: Although most (>=70%) of the available information `prt_val` is correct but we do not know the positions, so we form the constraint same as above for every entries, i.e.,
```
prt_val[i,j] - error <= eqn2 [i,j] <= prt_val[i,j] + error
```

Next we update the array `S` using `update()` function and again form the constraints using updated array `S` in the same manner as above, according to the case: `Case 1 or Case 2`. Continue this upto `N` times. Lastly feed it to the SMT solver and extract the solution to check whether we are able to recover `array_org`.

## Output 
If the given set of constraints is `satisfiable (sat)`, then we compare the obtained solution with `array_org`. In the case if it does not matches, it means multiple solution exists. Instead of exploring all solutions, we chooses to increase `N`, which in turns increases the number of constraints.

## Observed Problems
For parameters `array_size = 64; prt_len = 16 ; error = 2; N = 60`, we observed that in `Case 1` the model is solvable in `10-20 seconds` whereas 
in `Case 2`, the model is not giving solution even after long time irrespective of the fact that no. of errors in `Case 2` is very less as compared to `Case 1`. Note that for smaller set of equations, this time difference is not detectable.  
