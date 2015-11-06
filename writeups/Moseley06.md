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

### Chenggang Wu

The paper states at the beginning that complexity is the major difficulty in developing large-scale software systems. Then it summarizes the causes of complexity, which include (mutable) states, controls, and code volume (depend on the state and control). It analyzed three types of programming models, namely object-oriented programming, functional programming, and logic programming, and analyzed the complexity of these models based on the three causes and their trade-offs. In the second half of the paper, it proposes functional relational programming model, which cleanly separates essential state, essential logic, and accidental states and controls. The paper appraises this programming model that it achieves low complexity by minimizing the three causes.

I am interested in the trade-off between complexity and performance. It seems that the paper advocates the elimination of accidental states to reduce the complexity of the program. However, to improve the performance, it might be a good idea to add some accidental states and controls. Is there a way to simultaneously achieve high performance and low complexity?


### Johann Schleier-Smith

I find the premise of this paper to be extremely important, and interesting, and I find many of the ideas to be resonant with my own experience. The notion that “simplicity is vital,” that it beats testing, formal reasoning, or informal reasoning is a compelling one, one that shows us how to think about building better software systems. Said another way, “elegance is not optional,” or better yet—software should be elegant—it's a principle I can believe in.

The authors spend a great deal of time analyzing the antithesis of elegance and simplicity, which is complexity. They assert that complexity comes from the challenges of reasoning about state, be that formally or informally, or in seeking understanding through testing. Again and again, the authors emphasize how limited our ability to generalize becomes when state is involved. Again and again, they emphasize that understanding one state tells us “nothing at all” about other behaviors. This repeated assertion seems reasonable, but it is something that I would have liked to see further detailed and defended.

In their analysis of complexity, the authors identify *essential complexity* and *accidental complexity*, distinguishing between the two in terms of the user's problem. Importantly, essential complexity involves what is observable about the system, namely its input and outputs. This means that control flow becomes entirely a matter of accidental complexity, whereas some aspects of state are essential complexity, and others may be accidental complexity. That there is not essential control was a new recognition to me. Furthermore, though the authors do not put in these terms, we can deduce that the essential state of the system is completely specified by a log of its inputs. I wonder whether coupling the notion of time with the notion of essential state may make the concept even more powerful, perhaps by providing a bridge to the notion of control.

In speaking to implementation, the authors propose a Functional Relational Programming (FRP) style. The relational model allows capturing of essential state in its purest form, while the functional approach helps to manage the accidental complexity, which though not essential to the problem is nonetheless necessary for providing a practical solution. There is a lot to like about these ideas, and I wonder what more we have learned in nine years since the publishing of this paper, what views have been reinforced, what views have been undermined, and what is keeping us from realizing a world with mostly elegant software.