---
subheadline: "Sentiment Analysis"
title: "The Positivity of Ministers Tweets"
teaser: "A sentiment analysis of ministers and shadow ministers tweets."

catagories:
  - policy

tags:
  - sentiment analysis
  - AI
  - politics
  
header: no
---

<style type="text/css">
.main-container {
  max-width: 600px;
  margin-left: auto;
  margin-right: auto;
}
body {
  font-size: 18px;
}
</style>


| party  | positive |     n |  prop |
|:-------|:---------|------:|------:|
| Labour | 0        | 10290 | 40.24 |
| Labour | 1        | 10785 | 42.17 |
| Labour | NA       |  4498 | 17.59 |
| Tory   | 0        |  3993 | 17.07 |
| Tory   | 1        | 14927 | 63.83 |
| Tory   | NA       |  4467 | 19.10 |


![party mood]({{ site.url }}sentiment-plots/party-mood.png)

Boris Johnson is consitently more positive than Keir Starmer. But this
is really to be expected. In some ways Boris Johnson uses twitter in a
different manner to Keir Starmer. Boris generally uses it in the context
of his position in the Government, reporting the actions of his
Government and painting those actions in a positive light. Keir Starmer
on the other hand is spending his time trying to convince people that
the Government is doing a bad job at running a country and that people
should vote Labour in the next election.

Comparing main ministerial departments to their shadow counterparts



![cabinet mood]({{ site.urlimg }}sentiment-plots/cabinet_sentiment.png)



![scandals]({{ site.urlimg }}/sentiment-plots/scandals.png)



![popularity]({{ site.urlimg }}/sentiment-plots/popularity_model.png)
Perhaps this says more about Labour’s twitter followers than it does
about the effectiveness of negativity for the Labour party’s popularity.
<br/><br/><br/><br/>
