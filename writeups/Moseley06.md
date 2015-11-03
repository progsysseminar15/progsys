#Out of the Tar Pit
##Ben Moseley, Peter Marks

###Xinghao Pan
The authors argue that much of the complexity we see in large-scale software systems results from overly powerful languages that require programmers to over specify state and control of their programs.
All the control and most state are not essential to the problem, that is, they do not pertain to *what* we desire, but only to *how* it is being solved.
To better manage the complexity, the authors advocate avoiding all accidental and useless state and complexity, and separating the remaining useful parts into
- Essential state: The foundation of the system, which cannot reference other parts of the system. State is defined using relational logic.
- Essential logic: The heart of the system, which specifies the problem and desired outcomes, and can only reference the essential state. Integrity constraints are defined here using functional, relational language.
- Accidental state and control: This contains the additional stuff (declared declaratively) that helps improve the performance, but is not required to ensure correctness of the program.

This separation helps the programmer to reason about his problem, since he can work his way up from essential state to essential logic to the accidentals, without having to think about the logic of the later components.
Each component can be defined using different languages suited for their purpose.
In particular, the authors suggest using restricted languages that provide only enough power, since the more restricted a language, the easier it is to reason about.
More concretely, the language chosen by the authors is a "functional, declarative programming" language.

Two related takeaways I had from reading this paper were
- Powerful languages force complexity on the programmer. Restricted languages are easier to reason about.
- Not all formal specifications are executable, and not all executable specifications are efficient.
Taking these ideas further, additional restrictions on the specifictions (logic) can lead to a subset of programs that have even more efficient executions.
These restrictions on the logic can be supported by corresponding restrictions on the programming language, so that we end up with a language for efficient programs that is easy to reason.
As an example, monotone logic is supported by (monotone) Bloom.
Similarly, progressive systems (with potentially non-monotone operations) can be supported by a language like CQL.
It would be interesting to figure out what classes of logic we can efficiency express and execute.
(Since we argue that *all* systems can be thought of as progressive systems evolving over time, are there other interesting classes of programs between monotone and Turing-complete?)
