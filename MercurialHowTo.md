creating initial local copy:
```
hg clone http://hg.sympy.org/sfepy
```

using http to pull changes to local copy:
```
hg incoming http://hg.sympy.org/sfepy
hg pull http://hg.sympy.org/sfepy
```

using ssh to push local changes to the repository:
```
hg outgoing ssh://hg@hg.sympy.org/sfepy
hg push ssh://hg@hg.sympy.org/sfepy
```