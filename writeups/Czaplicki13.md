# Asynchronous Functional Reactive Programming for GUIs
## Evan Czaplicki and Stephen Chong

### Johann Schleier-Smith

Reactive programming has in recent years captured a great deal of mindshare in industry. This work describes theoretical foundations supporting the movement, provides a clean illustration of Functional Reactive Programming (FRP) in a simple language, and highlights specific performance challenges.

The Elm core language is quite small, which goes to show that by choosing building blocks carefully we can achieve a great deal of expressiveness. While I don't have a proof that the declarative nature of the languages is the key to its simplicity and power, I would advocate that this a key underlying reason.

Elm has an interesting approach to history. While it allows the programmer to define state variables as folds over history, its type system disallows certain compositions of these types, ensuring that it never needs to store any history. I am curious to understand this better, especially from the progressive systems perspective. I wonder whether this limitation is, as the authors claim, not a problem in practice, or whether it is an obstacle in certain useful programs. I also wonder whether a runtime implementation that understands history might do without the async language features, whether it might on its own guarantee a sort of eventual consistency, extracting concurrency implicit in the program and reshuffling execution to optimize responsiveness. This seems like a fruitful area for further research.