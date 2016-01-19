## 1 Introduction
* this material applies to Java 7 update 40 and Java 8 initial release
* enable JVM flag: `-XX:+FlagName`
* disable JVM flag: `-XX:-FlagName`
* setting a parameter for a flag: `-XX:FlagName=something`
* flags have defaults which depend on platform and other command-line arguments to the JVM
* *ergonomics* - process of automatically tuning flags based on the environment
* classes of machines: "client" and "server" - depends on environment
* performance of Java code depends on:
  * writing good algorithms
  * writing less code (the more code the longer compilation will take, more classes to load, more garbage collecting etc.)
  * optimize code early: Donald Knuth "We should forget about small efficiencies, say about 97% of the time; premature optimization is the root of all evil." means to write clean straightforward code that is simple to read and understand. Don't change algorithm and design to more complicated before profiling the program and being sure that they are really needed. However avoid constructs that are know for bad performance. Every line of code involves a choice, and if there is a choice between two simple, straightforward ways of programming, choose the more performant one. Example: logger and string concatenation done even when logger will not write this string to file/console.
  * look at other components: database, application server, backend server etc.
  * optimize for common case:
    * use profiler and focus on operations taking the most time
    * consider more likely bugs first: performance bug in new code > configuration of machine > bug in JVM or operating system
    * optimize what is used the most by users

## 2 An approach to performance testing
* FIRST PRINCIPLE: testing should occure on real product in the way the product will be used
  * categories of code used to do performance testing:
    * microbenchmarks - a test designed to measure a small unit of performance, e.g. single method invocation;
      * not easy to implement properly - compiler aggresive optimizations can change code so much that we're not measuring the right code (or no code at all);
      * also microbenchmark code can change production code multithreading behaviour and produce false statistics;
      * microbenchmarks must include a warm-up period - otherwise they will measure compilation time instead of only code performance
      * input values should be precalculated because otherwise benchmark will measure data preparation and produce false statistics;
      * it's result depends on input values range; random range isn't always the case; on the other hand common case input values may give false overall performance info;
      * measuring nanoseconds periods - is it worth to optimize rarely used nanoseconds code? Limited times when microbenchmarks are useful.
    * nanobenchmarks - measured whole application in conjunction with external resources (LDAP, database etc.) it uses
      * complex systems behave differently when all parts work together in contrary to mocking these parts;
      * many aspects of the JVM are tuned by default to assume that all machine resources are available, but performance behaviour may be different when other applications also run on this machine; example: when app uses 40% of CPU at average it means that most of the time it uses 30% and when GC is on the CPU usage goes up to 100%. When there are other apps running on the same machine our app won't have CPU time capacity like when it was single and will perform measurably different;
      * testing an entire application is the only way to know how code will actually run.
    * mesobenchmarks - somewhere between; they do some real work but not testing the whole application; it's measuring isolated performance at modular or operational level; reasonable compromise (but not substitution) for testing the full application
* SECOND PRINCIPLE: performance testing involves various ways to look at the application's performance
  * elapsed time (batch) measurements - the simplest way of measuring performance is to see how long it takes to complete a certain task
    * application performance changes in time due to application warm-up, done by:
      * JIT
      * filling caches in Java application
      * filling caches in database application
      * filling caches in operating system
      * warm-up in other places
    * application warm-up may take lots of time (45 minutes or more for some application servers configurations) so performance during this time may be important for users
  * throughput measurements
    * usually reported as operations per second (OPS), transactions per second (TPS), requests per second (RPS)
    * in client-server testing the client has no think time - sends request immediately after receiving response
    * in client-server testing the client has to be fast enough to really test the server; amount of client work depends on number of threads and how much they're doing; throughput measurement client uses less threads than response time measurement
    * it's common that these tests report throughput and average response time; 500 OPS x 0.5 s performs better than 400 OPS x 0.3 s
    * typically measured after warm-up