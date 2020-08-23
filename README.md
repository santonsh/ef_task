
# Trading home task

The task is to write an automated strategy that reads a stream of timestamped ask/bid quotes of two assets, and outputs a series of trades in order to generate profit.

### Files included

data.json - includes timestamped ask/bid quotes of two assets.
output.json - includes an example of a series of trades in the required format.
evaluator.js - a nodejs script to evaluate output files and calculate their profit. (You can either run it, or review it to understand how the evaluation of trades is calculated).

### Task

Write a strategy to generate a trading series. You can use the evaluator to test it (with `node evaluator.js`) or write your own in whatever language. Consider:

* Treat data.json as a stream to be process in real time - i.e. every trade you 'execute' for a timestamp should be a result of the data the preceded that timestamp.
* To simulate the changing order book, trades should not be within 30 seconds of each other. (The evaluator will throw an error in case of a violation).
* Assume that every order is of size 1.

On top of the resulting process, add your thoughts of the following points:

* What are the differences between running such a trading simulation and real world trading?
* How would you improve your strategy given more time/resources?
* How would the strategy differ given three assets? Four assets? N assets?

