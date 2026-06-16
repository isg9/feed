---
title: Prediction Markets Redux
url: https://blog.computationalcomplexity.org/2026/05/prediction-markets-redux.html
published: "2026-05-14T13:15:29Z"
feed: complexity
guid: tag:blogger.com,1999:blog-3722233.post-284448375668823016
---

# Prediction Markets Redux

For those very long-time readers this blog [extensively covered prediction markets](https://blog.computationalcomplexity.org/search?q=prediction+markets) from 2006 to 2008. In a prediction market, you have a future event, such as the winner of an election, and a market that pays off one dollar if that event happens and nothing if it doesn't. The price of this market corresponds to a predicted probability that the event will happen.

I worked with David Pennock and Yiling Chen to create an interactive map that colored every state based on how likely it was to go Democratic or Republican for both the presidential, gubernatorial and Senate races.

[![](https://blogger.googleusercontent.com/img/a/AVvXsEhxM4IB2O2ZbyVtXSWTEMiOMF7jB3gqDl4owFCgXWGo5WjqRcdBi-kIXvJuYCFjICyZ2aHzCu6sjTJ3JYvzcBUYkHJk0RAXsFatMuxc78SrNYIBz83pWal2LZyMclDdi52BdKWIgWYtnJF3Mvj3n1sHGFW3SqBUuRmjAE0G9_90kZCiQqwj3hal)](https://blogger.googleusercontent.com/img/a/AVvXsEhxM4IB2O2ZbyVtXSWTEMiOMF7jB3gqDl4owFCgXWGo5WjqRcdBi-kIXvJuYCFjICyZ2aHzCu6sjTJ3JYvzcBUYkHJk0RAXsFatMuxc78SrNYIBz83pWal2LZyMclDdi52BdKWIgWYtnJF3Mvj3n1sHGFW3SqBUuRmjAE0G9_90kZCiQqwj3hal)

This all ended when Intrade, the source of the data that we used, [went out of business](https://blog.computationalcomplexity.org/2013/03/goodbye-old-friends.html) after its CEO died climbing Mount Everest and may have embezzled money from the company.

In 2016 prediction markets went out of favor after badly predicting against Brexit once and Trump twice.

But what's old becomes new again. Prediction markets are back with a bang, with [Kalshi](https://kalshi.com/) and [Polymarket](https://polymarket.com/) providing real money markets open to U.S. citizens, something we didn't legally have, except for very low stakes, twenty years ago.

Prediction markets play two distinct roles:

1. As a betting system so people can put their "money where their mouth is" based on their beliefs.
2. As an information aggregation system, to use the [wisdom of the crowds](https://blog.computationalcomplexity.org/2004/08/wisdom-of-crowds.html) to predict future events.

Sometimes these roles conflict with each other. With real money comes real greed and we've seen some recent issues of insider trading, most notably a US soldier who [used classified knowledge](https://www.justice.gov/opa/pr/us-soldier-charged-using-classified-information-profit-prediction-market-bets) about the then upcoming raid in Venezuela to net $400K on Polymarket.

Insider trading will likely lead to better predictions, but those who have bet on that market might feel cheated. In the short run, insider trading might be a good thing, but in the long run, if we scare away participants because they're afraid others who have more information will take advantage of them, then we have fewer people in the market making it harder to draw upon the wisdom of the crowds.

Another challenge for prediction markets is determining how the market pays off. You need to come up with a very specific definition of what it means for the market to pay off, but strange circumstances can lead to unexpected results. Back in 2006 North Korea launched a test missile outside the country but the [market for that event paid off zero](https://blog.computationalcomplexity.org/2006/08/predictions-markets-mess.html) because the U.S. Department of Defense never verified that the missile actually left North Korean airspace.

The market I like to see is factoring the [RSA numbers](https://en.wikipedia.org/wiki/RSA_numbers) by a given date. Very easy to verify. Some people might buy such a security, expecting quantum computing to be factoring numbers soon. And then I can make some money by betting against it.
