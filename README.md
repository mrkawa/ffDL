# Fantasy Football Deep Learning

## Purpose

The aim of the project is to apply deep learning techniques appropriate  for time series analysis to generate projections for fantasy football points for a given player in a given week.

## Data

The dataset for this project was collected using the [nflfastR](https://github.com/mrcaseb/nflfastR) API, which scrapes play-by-play data for all NFL games dating back to the 1999 season.  In order to adapt for the purposes of this project, the data was grouped by player and by game and aggreggated into a more useful dataset.

## Features

The dataset provides hundreds of insightful features, many of which are more useful for in-game analysis and strategic decision-making than for week-to-week, season-to-season analysis, though many are applicable to the latter or even both.  For the purposes of this project, a custom feature was created to represent fantasy points as a simple linear combination of appropriate inputs following a standard fantasy football scoring system.  This fantasy points feature, in addition to existing at the player/game level was also aggreggated based upon the opponents the players were facing in a given week such that for example, the total number of fantasy points allowed by the New York Jets--to each position--in each respective week, is easily accessible as a time series of its own.

## Model

The model chosen for development towards the aforementioned goal was a Long Short-term Memory (LSTM) deep neural network, a specific form of Recurrent Neural Network (RNN).  Where RNNs *recursively* pass an ever-changing output state around back to itself as input and learn from its changes over time, LSTMs operate similarly, but with the additional introduction of "forget gates" to control what information is retained and what is "forgotten" each iteration of the network.  This is useful in mitigating the "vanishing gradient" problem where information from too long ago in the network's memory can cause internal weights to not be updated by a sufficient amount for the importance of the information.

## Implementation

The model was trained solely on one player's data at a time, being fed two parallel time series: their own fantasy points scored and a rudimentary projection of how many fantasy points their opposing defense would be expected to allow each weak.  The latter was included as a general estimate of strength to act as a basis of comparison to contextualize the player's actual performance against such (i.e. if a player is going against a stronger defense in a given week, it might make sense to expect him to score fewer points, and vice-versa).

For the purposes of experimentation, a few variables were held constant to maintain consistency and comparability of results; many of these are likely sub-optimal and the model would benefit greatly from their tuning.  Firstly, the model was trained on a moving window 16 games (1 full season) in length.  (As an aside: this is not ideal as it (a) requires ignoring the post-season and (b) assumes every player plays a full game every week and never sits out due to injury or lineup changes.)  Additionally, training was performed for 12 epochs, as that was shown to be generally the approximate point when training and validation loss began to diverge (in other words, the model stopped fitting and started overfitting).

Evaluation of the model was done on the most recent 16 game window of their available data (which was of course withheld from the training data) and accuracy was measured using a simple Mean Squared Error.

## Results

Comparing the model's next 16 projections to the actuals for their most recent 16 games, it was clear that the model was generally quite decent, if a bit conservative.  This is not of great concern since fantasy performance in the real world is a product of so many eclectic factors, many of which outside of the player's control or worse, completely random.  What this means is that actual data will naturally display very high variance from a baseline performance level/trend.  Such variance per se is extremely difficult to predict and any attempt at such would very likely need to take into consideration far more exogenous variables than simply expected strength of opponent.  So ultimately, predicting the player's baseline performance with some degree of reasonable-ness ought to be considered a success within the time and resource bounds of this endeavor. 

## Future Steps

As just aforementioned, incorporating more exogenous features, both time series and static in nature, could greatly benefit with predicting week-to-week swings up or down in players' performances.  Factors such as performance of teammates over time, location, time zone, weather, injury status, etc. all conceivably have significant impacts on performance.  Additionally a benefit of considering such factors is to enable somewhat personalized projections for players without enough/any historical game data of their own yet.

Another consideration that was made in the project was to retrain a new model completely for every individual player based on his own history.  Perhaps a model trained on all players could benefit from the vastly larger dataset available and capture insights not clear from limited training sets.  Perhaps instead each player is sufficiently unique in their trajectories and behaviors that the model would suffer from a lack of personalization as well, but the only way to find out is to experiment.

More development should ideally be put into the loss function used to train the network as well.  Foremost because MSE is particuarly sensitive to outliers and, if anything, it should be particularly forgiving towards due to the randomness involved.  But also to capture any other quirks of the domain, perhaps a more custom solution would be appropriate.

A separate LSTM for generating more accurate "fantasy points allowed" projections for the players' opponents would almost certainly serve its purposes better than the rolling average method implemented.  And finally, more hyperparameter tuning should be conducted to improve performance no matter what the ultimate, refined model setup turns out to be.
