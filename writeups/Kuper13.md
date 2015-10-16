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


### Johann Schleier-Smith
The authors describe extensions to a lattice framework that some of the authors had previously developed, extensions that promise to make the technique more usable and practical. I gained some further perspectives on the use of lattices for distributed computation, a notion that I had been familiar with but had not had the chance to think through deeply. What strikes me as particularly thought-provoking and novel in this paper is the notion of “quasi-determinism,” the relaxation of the notion that the computation must always produce the same result by allowing for failure. This strikes me as similar to the notion of optimistic concurrency control, where we allow distributed computation to proceed knowing that it may not always succeed.

I am not familiar with Haskell so following some of the code examples in this paper was a bit of a challenge for me and I'm sure I'm missing important implementation details. I continue to wonder how practical the LVars approach is, even with freezing and event handler extensions. Is it straightforward to translate from a programmer-friendly format into this execution model? Do such programs execute efficiently?

### Chenggang Wu

In the world of parallel programming, non-determinism is often the enemy of programmers’ as non-deterministic programs may introduce non-reproducible bugs that are hard to detect, reason about, and fix. In order for the program to be deterministic, we should make sure that concurrent tasks must be oblivious to the effect of scheduling, and this restriction should be enforced at either the language or the run-time level. LVars is an example of such languages: it uses monotonic shared data structures and ensures that the orders in which tasks modify the shared data structures are not observable. However, since information is constantly being accumulated on the monotonic data structure, any read query asking whether the LVar lattice has reached a certain threshold must return ‘yes’, otherwise the scheduling order will be leaked. As a consequence, the reading thread is blocked until the threshold has been reached. The key contribution of the paper is that it enables monotonic data structures to actually say ‘no’ by adding a ‘freeze’ primitive to LVar. Once LVar is frozen, any further write to the monotonic data structure will throw an exception, and at this point we know that no more writes will occur. Therefore, LVar can now safely respond to the reading thread with negative information.

I think ‘freezing’ in LVar is similar to the punctuations in streaming literature in the sense that intuitively, they both mark the end of some inputs. It would be good to investigate further about the differences in terms of their purpose.

