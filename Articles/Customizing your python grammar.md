
Did you know that you can add keywords and alternate spellings to your version of python?

To do so, clone the CPython repository (https://github.com/python/cpython.git) and follow the installation instructions (if you have all the libraries installed, it basically boils down to running  `./configure` then `make`) 

If like me you always type lamda instead of lambda, 
Go to Grammar/python.gram, look for 
```
lambdef[expr_ty]:
    | 'lambda' a=[lambda_params] ':' b=expression { _Py_Lambda((a) ? a : CHECK(_PyPegen_empty_arguments(p)), b, EXTRA) }
```

and replace `'lambda'` with `('lambda'|'lamda')`

Once this is done, you can run `make regen-pegen` to regenerate the grammar and then `make` to recompile python. 

Congratulations, you can now use lamda instead of lambda. The line below will work: 

`./python -c "print(sorted([(1,2), (2,1)], key=lamda x: x[1]))"` 


Even better, if you find typing lambda too long, you may want to use the $ symbol instead (or any one char litteral)

Go to Grammar/Tokens file and add the following line: 
DOLLAR                    '$'

and then in the python.gram file, replace 'lamda' with '$'`

You will need to run `make regen-token` to regenerate the tokens and then follow the same steps as above.

You can now use $ instead of lambda

`./python -c "print(sorted([(1,2), (2,1)], key=$ x: x[1]))"` 

Note: this example is heavily inspired by the book CPython Internals by Antony Shaw. 



