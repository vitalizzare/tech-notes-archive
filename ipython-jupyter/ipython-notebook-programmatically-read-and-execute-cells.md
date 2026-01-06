# Question

In an IPython Notebook, I would like to programmatically access and execute other code cells from within a code cell.

For example:

```python
if condition:
    # execute code in cell 3
```

I found a solution [here](http://nbviewer.ipython.org/gist/minrk/5491090/analysis.ipynb), where the function `execute_notebook` reads an `.ipynb` file cell by cell and executes the code cells using `get_ipython().run_cell()`.

Is there a way to achieve the same behavior without reading cells from an external `.ipynb` file? More generally, is it possible to write macros or otherwise reference and execute existing code cells from within an IPython Notebook itself?

# Answer

I believe that in this case it would be better to wrap the cell's code in a standalone function and call it as needed, rather than re-running the cell itself. However, as a thought exercise, here are possible alternatives:

1. Execute the cell's code using `exec`, accessing the cell by its number via the `In` dictionary, see [Input caching system][1]:

   ```python
   if condition:
       exec(In[3])    # reuse the third input cell
   ```

2. Use marked cells together with the [`rerun` magic command][2], making sure that the cell to be rerun is marked with a unique string:

   ```python
   if condition:
       %rerun -g 'unique mark'
   ```

3. Use the [`macro` magic command][3] to create a global name that refers to the code from the several specified cells joined together. This name acts like an automatic function which re-executes those cells as if you had typed them. 

Example:

```python
In [1]: x, y = 10, 20

In [2]: # sum of x and y 
   ...: xy_sum = x + y
   ...: print(f'The sum of {x=} and {y=} is {xy_sum}')
The sum of x=10 and y=20 is 30

In [3]: # max of x and y
   ...: if x > y:
   ...:     xy_max = x
   ...: else:
   ...:     xy_max = y
   ...: print(f'The max of {x=} and {y=} is {xy_max}')
The max of x=10 and y=20 is 20

In [4]: x, y = 1, 2

In [5]: # Repeat the second cell
   ...: exec(In[2])
The sum of x=1 and y=2 is 3

In [6]: # Repeat the most recent cell containing 'max of x and y'
   ...: %rerun -g 'max of x and y'   
=== Executing: ===
# max of x and y
if x > y:
    xy_max = x
else:
    xy_max = y
print(f'The max of {x=} and {y=} is {xy_max}')
=== Output: ===
The max of x=1 and y=2 is 2

In [7]: %macro -q max_and_sum 3 2

In [8]: x, y = 7, 5

In [9]: max_and_sum
The max of x=7 and y=5 is 7
The sum of x=7 and y=5 is 12
```

That said, writing a function and reusing it is still the recommended and most maintainable solution.

  [1]: https://ipython.readthedocs.io/en/stable/interactive/reference.html#input-caching-system
  [2]: https://ipython.readthedocs.io/en/stable/interactive/magics.html#magic-rerun
  [3]: https://ipython.readthedocs.io/en/stable/interactive/magics.html#magic-macro

