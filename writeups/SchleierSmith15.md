# An Architecture for Agile Machine Learning in Real-Time Applications
## Johann Schleier-Smith.  

### Jane Student (example)
This paper's discussion of the semiotics of Hawaiian drinks is very different from the way we usually think about tropical beverages.  I particularly was struck by their meta-analysis of the self-negating blend of "hard" liquor and "pulpy" fruit. I wonder if this has implications for the hard-drinking heroes of pulp fiction.  Maybe we can apply this to Dashiell Hammett -- either to generate authentic posthumous texts, or to synthesize publishable academic analyses based on the extant texts.  If anybody would like to discuss these ideas for a course project come find me.

It almost goes without saying that the paper's user study was biased by the inebriation of the authors. But I actually believe the main observations are sound, and would withstand closer investigation.  My main question is not around validity, but  context -- if we shifted from Hawaiian drinks to Alaskan drinks, would the assumptions of the paper hold?  Is there an analogously soft Arctic mixer that plays the role of pineapple -- maybe the butter in a Hot Buttered Rum?  If I had to guess, I'd say that the lessons of this paper may well be limited to tropical climes.

### Erik Krogen

While there are some interesting takeaways in the paper about machine learning, I will focus on those which are more relevant to our course. The first is obviously the flexibility of the append-only EventHistory from the development point of view, making it very easy for data scientists to perform test runs on real data. One thing that stands out to me throughout their software engineering notes is the care they put into optimizing everything to work quickly and efficiently on a single machine to remove the necessity for cross-machine synchronization, which combined with replicas provide high availability.  

One open question I see pretty clearly remaining is just how large this can scale. Working at 10 million users is great, and will be sufficient for most businesses, but I imagine that at e.g. Twitter scale a single server would no longer be enough to handle the incoming traffic. It does seem that the ideas presented in the paper should still be applicable even if the online processing overloaded a single server and required the use of a distributed system. One thing I am curious about is how far back they are able to store events in the EventHistory repository. It seems that this repository must grow in size quite quickly, and I doubt that it would be able to serve events quickly enough if it was also able to serve arbitrarily old data. 

### K. Shankari
#### Overview
This paper presents a practical view, motivated by a real-world example, of
agile data modelling. I found it interesting to read a discussion of the system
building around a machine learning framework that did not focus on either the
machine learning algorithms, or the execution framework, but on the process for
building models.

#### Strengths
- The insights are based on a real-world system.
- The notion of collecting large amounts of read-only data in an event stream
  and running analyses on it, while fairly intuitive, has not been explored and
  evaluated prior to this.
- It is challenging to come up with metrics for such vague concepts as
  *agility* or *ease of use*, although they are very real barriers to adoption.
  The paper attempts to quantify metrics that could be used to study this in the future.
- The system takes into account that external *lookup* state can be modified,
  and explicitly captures this through *fact change events*. It was helpful as
  a reminder that the state of the system includes external dependencies, and
  in order to roll the world back, these dependencies need to be captured and
  logged in the event stream as well.

#### Weaknesses
I would have liked to see more discussion of the following topics:
- Versioning: The system described here works as long as all your data is in
  the same format. But what if you change the format of your data, possibly by
  collecting different information?
- Given the strong focus on consistent modelling between exploratory evaluation and
  production code, it is not clear why you chose two different programming
  languages for the two parts? Surely, using a langugage such as python that
  supports both ongoing development and modelling would allow you to unify the
  two parts even further.
- Some terms are loosely defined and explained. For example, what is training
  recommendation and how does it help?
- While the evaluation section is a laudable attempt for an engineering-focused
  paper, it is a bit primitive by academic standards. For example, do you know
  that the increase in matchers and voters is not just due to increase in your
  user base? Also, while Jul-Nov 2013 shows frequent model updates correlated
  with more voters and matchers, Nov 2013 - Feb 2014 shows an increase that is
  not correlated with model updates, and Feb - Apr 2014 shows updates that are
  not correlated with an increase. It would be good to provide some more
  insights into these changes.

###Chenggang Wu
Event history architecture is good for agile machine learning in real-time applications. The main reason is that it allows rapid experiment cycles by letting the developers jump back to any frame of history, train the model with new features, and test the effectiveness of new ideas by applying the trained model to the present events. Since the feature definitions and state management software are same in development and production, it releases the burden from the developers by letting them write one piece of code that fits for both production and development. Because of the reduced software engineering overhead, the abstraction makes it easier for multiple data scientists to merge their efforts and improve the model concurrently. Another interesting lesson is that devising interesting features might be more important than choosing the ‘right’ machine learning algorithms and parameters. Because of the low latency requirement for real-time applications, a good feature must also be quickly accessible.

Under the event history architecture, it would be interesting to see how the length of training intervals will affect the model’s prediction performance. Also, since developers can jump back to any frame of history, how do we know the optimal point that will give us the most promising model for predicting the future? Furthermore, now it seems that all the events are included in a single time interval and trained together. Does it make sense to split events into groups by certain characteristics (location, age, …), project each group of events to a different time interval and train each group separately?

### Gabe Fierro

Machine learning models take a long time to develop and push into production
for evaluation. An iterative approach ("agile" programming) is desired, but
rapid development is complicated by the lack of a uniform framework for running
models on historical data and then migrating implementations to real-time data
in production.  The paper presents an architecture for storing input data as a
time-ordered event history, which can then be treated as a playback log for
real-time testing or as point-in-time feature state for static model formation
("back-testing"). By providing the same API and semantics for creating models
in both testing/exploration and production, the feedback loop is optimized for
iterative development. Uniform representation of real-time and historical data.
Uniform access method because now everything is in the log.

How do models scale with larger features? Is a packed in-memory representation
still appropriate?  What happens when new features are introduced -- is there a
feedback mechanism to establish when a feature was introduced/decomissioned?
The paper asserts that handling >10k updates per second across 100+ features is
not resource intensive and obviates the need for distributed architectures, but
how would this work when scaled to 100million or 1 billion users or more update
requests? Are there heuristics that can be applied to reduce the number of computations
or cache intermediate results of iterative model training that can reduce computation
time? To reduce memory pressure for larger feature sets and larger data sets, how
can knowledge of the problem help drive cache management of the event log?

