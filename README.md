# AMFStockMarket
 ENS Data Challenge - What do you see in the stock market data? 
 URL : https://challengedata.ens.fr/participants/challenges/66/

 - Machine Learning Challenge proposed by ENS and the Autorité des Marchés Financiers 
 - Implementation of Machine Learning models (XGBoost, RandomForestTree, LSTM) to detect market anomalies from order book data on 97 financial products

## Context
To ensure that markets function properly, the AMF has developed a battery of algorithms that aims to detect atypical events among the tens of millions of market data, collected daily. There are various kind of atypical events: some of them are easily recognizable as sudden variation in prices or in volumes or even high volatility periods with oscillations in prices but others are more sophisticated.

In order to focus the challenge towards the detection of an atypical event occurrence, instead of the detection of one type of event, the challenge context will voluntarily not describe all the different kinds of events.

## Goal
The goal of the challenge is to train machine learning models to look for anomalies within stock market data.

It is relatively easy to design one algorithm seeking one specific kind of event, and then to implement as many algorithms as there are types of atypical events. However, it would be also beneficial to be able to detect any type of atypical events thanks to one model, which could learn to recognize common features between these different events.

From a sample of market data: time series price and volume data, the challenger is invited to predict the existence of an atypical event on an hourly basis by financial instrument. Any kind of approach can be experimented but the AMF is particularly interested in computer vision techniques on reconstructed time series plots if the challenger thinks that it is relevant.

## Data description
The dataset is stored in Excel files:

y_train.csv
a folder x_train_file containing one csv per financial instrument
a folder x_test_file containing one csv per financial instrument

The label file (y_train) indicates when we have detected an event on an hourly basis and by financial instrument. When EVENT is set at TRUE it means an event has been detected whereas when set at FALSE means there is no particular event found in the slot. An event can occur in a short time period (~15 minutes): in this case, the label file does not specify exactly when the event takes place in the hour. If several events appear in the same hour, the label file does not distinguish them and consider the case as if there is only one event during the hour. When the label file indicates multiple events in the same day on the same financial instrument, the challenger should consider that these events are interlinked.

The train and test data contain both the equivalent of 1 month of data (train data are prior to test data) for 97 financial products. Train and test data contain the main information to gain a broad understanding of the market kinematic. The data corresponds to all the orders and trades linked to the financial products within the time period. In order to gain a better understanding of the order book, here are several ressources:

https://www.investopedia.com/terms/o/order-book.asp,
https://ec.europa.eu/finance/securities/docs/isd/mifid/rts/160624-rts-24-annex_en.pdf,
https://youtu.be/Kl4-VJ2K8Ik

Train and test data exhibit in their rows the same time series for a given trading date t on a certain financial instrument a:

 - 1 order id unique
 - 2 timestamp
 - 3 sequence number (to sort the different messages that could occur at the same exact timestamp)
 - 4 type of interaction:
NEWO/TRIG = submission (TRIG is when an order becomes active and visible after a special event, to simplify things for the challenge, can be considered as a new submission)
REME/REMA = modification
CAME/CAMO = cancellation
PARF/FILL = execution of an order (a transaction - trade - occurs), PARF if order is partially filled and FILL if order is fully filled
EXPI = expiration (meaning when an order is no longer valid, it expires)
 - 5 order type - there are different types of order, the information can help the challenger to understand the data kinematics but not essential for the challenge (https://www.investor.gov/introduction-investing/investing-basics/how-stock-markets-work/types-orders, and https://www.investopedia.com/terms/i/icebergorder.asp)
 - 6 if the variable #4 is equal to PARF/FILL, then the side (Buy or Sell) that triggers the transaction is filled with AGRE, otherwise the field is blank
 - 7 side of the interest concerned by the message, B = buying order and S = selling order
 - 8 price of the order (i.e. price up to which the market participant is willing to buy or sell) - if the price is equal to 0, this is not an error, in this case it means that the market participant want to buy or sell a security immediately at the market price (see #12 and #14)
 - 9 quantity of the order (i.e. number of shares that the market participant wants to buy or sell at the specified price #8)
 - 10 if the variable #4 is equal to PARF/FILL, then the price of the transaction is filled, otherwise the field is blank
 - 11 if the variable #4 is equal to PARF/FILL, then the quantity of the transaction (i.e. number of shares) is filled, otherwise the field is blank

The following fields provide aggregated information about the existing interests of buyers and sellers (the best selling/buying prices and the available quantity immediately tradable):

 - 12 the best ask price at which the market participants are ready to sell immediately
 - 13 the total quantity (in number of shares) that can be immediately sold at the best ask price #12
 - 14 the best bid price at which the market participants are ready to buy immediately
 - 15 the total quantity (in number of shares) that can be immediately bought at the best bid price #14
The column #16 is redondant with the #4 type of interaction, it indicates when there is a trade (meaning when #4 is equal to PARF/FILL)
 - 17 trading date

Since a transaction is a match between one buyer and one seller, in the data, a trade is split into two rows (one for the buying order and another for the selling order that are matching). The two rows must share the same transaction price #10, the same traded quantity #11 and the same timestamp #2 and sequence number #3.

Timestamps between trades show period when buyers and sellers do not meet each other on a transaction price but the evolution of #12 the best ask price and #14 the best bid price provides information about the market trends as well.

When there is no data for a certain financial instrument on a specific date in x_train_file or x_test_file, it means that there are no transactions. In this case you should only find EVENT set at FALSE in the y_train_file or in the y_test_file.
