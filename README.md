# Applying-ML-to-existing-algorithms
How to integrate a machine learning layer into existing algorithms

This repository is an example of how I integrate a python machine learning model as an extra layer of decision making in a strategy that runs in Ninjatrader. I'm not providing my Ninjatrader algorithms or my machine learning algorithms, just the basic concept of how they can be connected.

My Ninjatrader strategy runs on the 1 minute time timefame and will take longs and shorts and is successful but I feel it could be refined more with something like along the lines of a trend indicator. However I had no success with traditional trend indicators. 

I did increase the profit factor  when I combined a machine learning prediction about the 15 minute chart direction. I used a linear regression/KNN model to predict the major trend direction changes and wrote the prediction to a .csv file. The ninjatrader strategy checks the .csv file as part of it's entry conditions and only enters in the ML prediction direction.
 
