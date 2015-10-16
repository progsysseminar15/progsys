#Freeze After Writing: Quasi-Deterministic Parallel Programming with LVars
##Lindsey Kuper, Aaron Turon, Neelakantan R. Krishnaswami, Ryan R. Newton

###Xinghao Pan
This paper addresses a problem with lattices -- since data can always be asynchronously added to the lattice, it is impossible to obtain a snapshot view of the lattice.
Freezing is an operation that declares that there are no further puts that will be added to the lattice.
Formally, this is done by extending the original lattice by a bit indicating whether the lattice has been frozen; any merge with a frozen lattice goes to a special top error element.
This enables a view into the final state of the lattice -- if we freeze it correctly, i.e. after all puts have been added, we will get a deterministic result; otherwise we will get an error.

There appears to be a close connection with punctuations, which are also statements that no further additions will be observed.
Both freezing and punctuations are non-monotone: the point at which the freeze or punctuation is done affects the final result.
The Bloom^L paper did consider an interesting variant for the shopping cart example though.
We can retain monotonicity if we can send a punctuation with a list of all puts, for example, by checking out the shopping cart with a list of operations executed.
Being able to generate that list is in general non-trivial, but could be done in special cases.

An easy extension of freezing would be to perform `partial freezes', i.e. punctuations on predicates.
