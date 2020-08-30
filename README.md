
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

* [Anton - answers specific to marketmaking strategy]
* What are the differences between running such a trading simulation and real world trading?
    * [Anton] At real world trading we dont know the winning bid/ask and we need to "guess it" by putting orders close to the edge of the order book
    * [Anton] In the real world trading fees are non zero thus not for every spread the trading is profitable
    * [Anton] Prediction (and probabaly decision) should happen at t-delta - a bit earlier than an actual timesample arrives
* How would you improve your strategy given more time/resources?
    * [Anton] I would detect further falls/rises in market price to cut the risk of market making strategy
    * [Anton] Turn the algorithms from tick based to time based to account for tick non-uniformity
* How would the strategy differ given three assets? Four assets? N assets?
    * [Anton] Given limited funds I would choose big spread assets and optimize portfolio bases of assets risk/profit predictions

* [Anton - general answers]
* What are the differences between running such a trading simulation and real world trading?
    * [Anton] Prediction (and probabaly decision) should happen at t-delta - a bit earlier than an actual timesample arrives
    * [Anton] Failsafe wrapper mechanisms should be implemented to protect against high losses due to buggy input and unexpected market behaviour
    * [Anton] The prediction model should be constantly updated and included continously in a bot continous backtesting on late data. This because market behaivour can change quite frequently on a daily/weekly basis
    * [Anton] For market making trading strategies (as I understand these) it can be a good idea to analyze input and account for competition: Number of players and their activity classification (as seen on zoomed in signals)    
    * [Anton] Multi assset portfolio target risk optimization should be implemented to make the bot more predicatable 
* How would you improve your strategy given more time/resources?
    * [Anton] Need to optimize calculation times and create appropriate efficient classes for predictors and strategy makers
    * [Anton] Need to account for tick non uniformity in data by creating new ticks (volume, total_sum, time, entropy based) or creating prediction models 
    * [Anton] Create new signals/features reflecting market/trading activity state (bullish/bearish score, volumes, active members score, etc)
* How would the strategy differ given three assets? Four assets? N assets?
    * [Anton] All the infrastructure - predictors, portfolio return calculators should be acomodated to use N layer data [kinda implemented]. Predictors should use cros signal correlation information
    * [Anton] Multi assset portfolio target risk optimization should be implemented to make the bot more predicatable and or profitable 


###########################################################################################################
###########################################################################################################
#################             Solution Notes            ###################################################
###########################################################################################################
###########################################################################################################

# Solution

### General plan of work
    ##################################################
    ################### V0 ###########################
    ##################################################
    * The work is perfrmed in a ipynb notebook
    * The first stage is to learn about signals. The next features were unalyzed:
        ** sampling frequency
        ** signal covariance
        ** base and moving average signal zoomout and zoomin exploration
    * The basic trading bot was implemented according to my humble understanding of trading bots
    * implement basic reference strategy for bot using "idealPredictor" as a signal predictor. idealPredictor just takes real data from the future so it is not a valid solution for a task strategy but can be used as a gloden standard for future predictors and strategies - [ONLY THIS STRATEGY IS IMPLEMENTED IN V0]
    * implement unfair overfitted NN predictor trained on the whole day data. Again it is not a valid strtegy but can be used for a debug and development purpoces and as a silver standard for next strategies - [NOT IMPLEMENTED IN V0]
    * implement fair NN predictor trained on the part of the day data and used to evaluate the rest of the day. This should implement a valid task solution - [NOT IMPLEMENTED IN V0]
    * implement dynamic NN predictor trained on the part of the day data and retrained constantly to imporove performance. A bonus strategy so to say. [NOT IMPLEMENTED IN V0]  

    ##################################################
    ################### V1 ###########################
    ##################################################
    * implement simple market maker strategy (what actually was required). [IMPLEMENTED IN V1] - [mm_strategy.ipynb]  
    

### Discaimer
0. Instead of reading about market making strategies "free wondering mind" approach was used and simple traditional bot was implemented in the version 0 (facepalm). After first version was submitted I've added a simple market making strategy as well. It is implementd in mm_strategy.ipynb  
1. A lot of effort was put into creating a bot instead of a strategy thus task target is still missed in v0 [but completed in v1]
2. Bot was implemented to trade in an "allA/allB/allFiat" manner orders instead of "orders of size 1" only

### Signals Study Conclutions
#### Signal sample frequency
1. the samples are highlty uneaven with rare time deltas up to several minutes. Most time deltas between the samples however are between 50 and 500 ms
2. Due to (1) several approaches can be taked to fight non uniformity of samples
    a. upsample missing timepoints
    b. craete new more uniform ticks by special transformation (time, volume, cumulative price ticks or entropy based ticks)
    c. train predictor in such a way that accounts for timme deltas between samples
    d. assume low impact of non uniformity and treat as uniform (probably a bad idea but will do for start)

#### Signal timebased dynamics 
1. Zoomout: 
    a. The trading dataframe is of one day aproximately
    b. A and B are hightly correlated at least at low resolution
    c. There are negative spreads in the data up to tens of seconds. Three of possible explanations are 
        i. Bullish of Bearish trades beyond optimal opposite price point 
        ii. missing data points/lags in an order book that creates opposite price optimization lag
        iii. Lagging price correction 

2. Zoomin
    a. Uneven sampling can be seen
    b. A and B have different trading dynamics
    c. A and B have different spread
    d. A and B have different trading patterns
    e. As a result of (b), (c), and (d) we can deduct different number of traders, hightly different volumes traded for assets A ndd B 

#### Var/Covar analysis
    a. Hmmmm... Not sure what can be seen here at low resolution in regard of var/covar between signals. 
    b. Peaks in A/B and bid/ask covariance correlates to occations of magor trading (volume, price change) likely (?)
    c. var and covar graphs for all the signals look alike which probably reflect the fact that assets are closely related. Especially it shows at major trading periods (spikes in var/covar)



### Trading bot
    1. Bot schema
    A simple trading system can be sketched as this:
    (A) marketInputs -> (B) features/signals -> (C) assetPredictions [t, price, prob] -> (D) portfolioOtimization/Strategy -> (E) orders

    * For (A) we have our initial signals

    * For (B) we can generate new features like trading frequency, volume, signal variance. In general case this can include outside signals like semantic market analysis or other exchanges data. 

    * For (C) possibilities are wast. The predictor can be short termed, long termed, tech analysis based, ML/NN based. In this assignment we will check 3 options for the predictor:
        ** a. Ideal predictor. The predictor that knows the future exactly. Built by using existing dataframe. It is good for initial debug and as a performance reference for other predictors
        ** b. overfitted NN predictor. We will train the NN on the entire dataframe and use it on the same dataframe in inference mode for trading. It is not a fair predictor as it uses information from the future for learning. However it can be used as a refeerence for future predictors
        ** c. NN predictor that is trained on a past window and is used for some time until the model get updated with new data. 

    * For (D) we will use a strategy that chooses to maximize predicted return on portfolio. For the sake of assignment we assume oreders of 1 only so we will not create risk-return optimized and distributed portfolios. In real world however it is a good idea to predict prices/signals/returns with probabilities and use them to maintain return-risk optimized portfolios be choosing assets to be close to efficiency frontier. 

    * (E) Orders should be of size of one every time and spaced at least 30 seconds each

    2. Predictor object
    Interchangable object that is responsible for calculating signals predictions. In our case without probabilities in first versions
    
    3. Return Predictor function
    Interchangable function that is responsible for calculating returns on portfolio or in our version for each asset based on data from predictor
    
    4. Decision Making function
    Interchangable function that is responsible for making decition based on calculated expected returns data produced by the former function



### Signal predictors and relating market taker strategies
    1. idealPredictor strategy. 
    Implemented and working. Output file is output.json

    2. overfitted NN predictor strategy
    Not yet implemented do to a lack of time

    3. fair NN predictor strategy
    Not yet implemented do to a lack of time

    4. dymanic NN predictor strategy
    To be implemented in future versions of the bot

### Simple market maker strategy
    * implemented in mm_strategy.ipynb
    * does not use market sharp fall/rise predictions
    * if market is relatively stable and there is a spread trades by realizing sells close to market ask and buys close to market bid
    * is not analyzed well
    