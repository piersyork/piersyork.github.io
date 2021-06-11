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

    # set rmarkdown options
    knitr::opts_chunk$set(echo = TRUE, warning = FALSE, comment = FALSE, message = FALSE,
                          fig.height = 5)

    # import packages
    box::use(readr[read_csv],
             dplyr[...],
             ggplot2[...],
             lubridate[date],
             forcats[fct_relevel],
             ggsci[scale_colour_lancet, scale_fill_lancet],
             plotly[ggplotly, layout, highlight],
             broom[tidy])

    # import data
    sentiment <- read_csv("Data/minister_tweets_sentiment_gl_nlp.csv")
    sentiment_date <- sentiment %>% 
      group_by(person, date) %>% 
      summarise(date, sentiment = mean(score, na.rm = TRUE), dept, party) %>% 
      distinct() %>% 
      ungroup() %>% 
      mutate(dept = fct_relevel(dept, c("Leader", "Cabinet Office", "Foreign Office", "Home Office",
                                        "Treasury", "Department of Health", "Ministry of Defence",
                                        "Department of Justice")),
             party = fct_relevel(party, c("Tory", "Labour")),
             positive = if_else(sentiment >= 0, 1, 0))

    sentiment_date_party <- sentiment %>% 
      group_by(party, date) %>% 
      summarise(date, sentiment = mean(score, na.rm = TRUE)) %>% 
      distinct() %>% 
      ungroup() %>% 
      mutate(party = fct_relevel(party, c("Tory", "Labour")))

    # set ggplot theme
    my_theme <- theme_minimal() +
      theme(legend.position = "bottom", legend.title = element_blank(),
            axis.title = element_blank(), plot.title.position = "plot",
            plot.caption.position = "plot", plot.caption = element_text(hjust = 0, size = 11))
    theme_set(my_theme)

    sentiment %>% 
      mutate(positive = ifelse(score >= 0, 1, 0),
             positive = ifelse(is.na(positive), "NA", positive)) %>% 
      group_by(party) %>% 
      count(positive) %>% 
      mutate(prop = round(n/sum(n) * 100, 2)) %>% 
      knitr::kable()

<table>
<thead>
<tr class="header">
<th style="text-align: left;">party</th>
<th style="text-align: left;">positive</th>
<th style="text-align: right;">n</th>
<th style="text-align: right;">prop</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: left;">Labour</td>
<td style="text-align: left;">0</td>
<td style="text-align: right;">10290</td>
<td style="text-align: right;">40.24</td>
</tr>
<tr class="even">
<td style="text-align: left;">Labour</td>
<td style="text-align: left;">1</td>
<td style="text-align: right;">10785</td>
<td style="text-align: right;">42.17</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Labour</td>
<td style="text-align: left;">NA</td>
<td style="text-align: right;">4498</td>
<td style="text-align: right;">17.59</td>
</tr>
<tr class="even">
<td style="text-align: left;">Tory</td>
<td style="text-align: left;">0</td>
<td style="text-align: right;">3993</td>
<td style="text-align: right;">17.07</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Tory</td>
<td style="text-align: left;">1</td>
<td style="text-align: right;">14927</td>
<td style="text-align: right;">63.83</td>
</tr>
<tr class="even">
<td style="text-align: left;">Tory</td>
<td style="text-align: left;">NA</td>
<td style="text-align: right;">4467</td>
<td style="text-align: right;">19.10</td>
</tr>
</tbody>
</table>

    sentiment_date %>% 
      # filter(dept == "Leader", date > date("2020-01-01")) %>% 
      filter(date > date("2020-01-01")) %>% 
      ggplot(aes(date, sentiment, colour = party)) +
      # geom_smooth(span = 0.4, se = FALSE) +
      geom_point(alpha = 0.3, size = 2.4, shape = 16) +
      geom_hline(yintercept = 0, colour = "black", size = 1, linetype = "dashed") +
      labs(title = "The Party's Mood",
           subtitle = "The sentiment of Tory ministers tweets and their shadow counterparts",
           caption = "Note: Sentiment is calculated using Google's natural language processing API, it \nestimates the overall positivity or negativity of a tweet") +
      scale_x_date(date_breaks = "4 months", date_labels = "%b %y") +
      scale_colour_lancet()

![](/Users/piers/Documents/GitHub/piersyork.github.io/_posts/twitter_analysis_files/figure-markdown_strict/b_k_plot-1.png)

Boris Johnson is consitently more positive than Keir Starmer. But this
is really to be expected. In some ways Boris Johnson uses twitter in a
different manner to Keir Starmer. Boris generally uses it in the context
of his position in the Government, reporting the actions of his
Government and painting those actions in a positive light. Keir Starmer
on the other hand is spending his time trying to convince people that
the Government is doing a bad job at running a country and that people
should vote Labour in the next election.

Comparing main ministerial departments to their shadow counterparts

    sentiment_date %>% 
      filter(date > date("2020-01-01")) %>% 
      ggplot(aes(date, sentiment, colour = party)) +
      # geom_smooth(span = 0.6, se = FALSE, ) +
      geom_point(alpha = 0.3) +
      geom_hline(yintercept = 0, linetype = "dashed", colour = "gray10") +
      labs(title = "The cabinet and their shadows",
           subtitle = "The sentiment of cabinet ministers tweets compared to their
    shadow counterparts") +
      scale_x_date(date_breaks = "6 months", date_labels = "%b %y") +
      facet_wrap(~dept, nrow = 2, ncol = 4) +
      scale_colour_lancet()

![](/Users/piers/Documents/GitHub/piersyork.github.io/_posts/twitter_analysis_files/figure-markdown_strict/facet_plot-1.png)

    mean <- sentiment_date_party %>% 
      filter(date > date("2020-03-01")) %>% 
      group_by(party) %>% 
      summarise(sentiment = mean(sentiment, na.rm = TRUE)) %>% 
      mutate(scandal = "Average")

    lockdown <- sentiment_date_party %>% 
      filter(date %in% date("2020-03-22"):date("2020-03-26")) %>% 
      group_by(party) %>% 
      summarise(sentiment = mean(sentiment, na.rm = TRUE)) %>% 
      mutate(scandal = "UK into \nLockdown")

    cummings <- sentiment_date_party %>% 
      filter(date %in% date("2020-05-23"):date("2020-05-28")) %>% 
      group_by(party) %>% 
      summarise(sentiment = mean(sentiment, na.rm = TRUE)) %>% 
      mutate(scandal = "Cummings \nLockdown Trip")

    gcse <- sentiment_date_party %>% 
      filter(date %in% date("2020-08-16"):date("2020-08-22")) %>% 
      group_by(party) %>% 
      summarise(sentiment = mean(sentiment, na.rm = TRUE)) %>% 
      mutate(scandal = "GCSE \nResults")

    patel_bully <- sentiment_date_party %>% 
      filter(date %in% date("2020-11-18"):date("2020-11-21")) %>% 
      group_by(party) %>% 
      summarise(sentiment = mean(sentiment, na.rm = TRUE)) %>% 
      mutate(scandal = "Priti Patel \nBullying")

    df <- rbind(mean, lockdown, cummings, gcse, patel_bully) 
    # df$scandal %>% unique() %>% paste(collapse = " ")

    df %>% 
      mutate(scandal = fct_relevel(scandal, c("Average", "UK into \nLockdown", "Cummings \nLockdown Trip", "GCSE \nResults",
                                              "Priti Patel \nBullying"))) %>%
      ggplot(aes(scandal, sentiment, fill = party)) +
      geom_col(position = "dodge") +
      labs(title = "The impact of scandals",
           subtitle = "How the sentiment of Tory and Labour tweets changes during \nGovernment scandals") +
      scale_fill_lancet() +
      theme(axis.text.x = element_text(size = 11))

![](/Users/piers/Documents/GitHub/piersyork.github.io/_posts/twitter_analysis_files/figure-markdown_strict/scandals-1.png)

    tidy_tweets <- read_csv("Data/tidy_ministers_tweets.csv") %>% 
      mutate(dept = fct_relevel(dept, c("Leader", "Cabinet Office", "Foreign Office", "Home Office",
                                        "Treasury", "Department of Health", "Ministry of Defence",
                                        "Department of Justice")))
    departments <- tidy_tweets$dept %>% unique()

    estimates <- list()
    for (i in 1:length(departments)) {
      model1 <- tidy_tweets %>% 
        filter(dept == departments[i]) %>%
        mutate(negativity = score * -1) %>% 
        lm(retweet_count ~ negativity*party + is_quote + hour + chars + is_retweet +
             subject_est, .) %>% 
        tidy()
      
      model2 <- tidy_tweets %>% 
        filter(dept == departments[i]) %>%
        mutate(negativity = score * -1) %>% 
        lm(retweet_count ~ negativity*fct_relevel(party, c("Tory", "Labour")) + is_quote + 
             is_retweet + hour + chars + subject_est, .) %>% 
        tidy()
      
      estimates[[i]] <- tibble(party = c("Labour", "Tory"),
                               dept = rep(departments[i], 2),
                               estimate = c(model1$estimate[2], model2$estimate[2]),
                               sd = c(model1$std.error[2], model2$std.error[2])) %>% 
        mutate(lower_conf = estimate - 1.96 * sd,
               upper_conf = estimate + 1.96 * sd,
               across(where(is.numeric), ~.x/10))
    }
    df <- bind_rows(estimates)




    df %>% 
      mutate(lower_conf = estimate - 1.96 * sd,
             upper_conf = estimate + 1.96 * sd,
             across(where(is.numeric), ~.x/10)) %>% 
      ggplot(aes(estimate, party, colour = fct_relevel(party, c("Tory", "Labour")))) +
      geom_point(position = position_dodge(width = 0.8), size = 1, show.legend = FALSE) +
      geom_linerange(aes(xmin = lower_conf, xmax = upper_conf), size = 0.8,
                      show.legend = FALSE, position = position_dodge(width = 0.6)) +
      geom_vline(xintercept = 0, linetype = "dashed") +
      facet_wrap(~dept, nrow = 2) +
      ggsci::scale_colour_lancet() +
      labs(title = "Sentiment and Popularity",
           subtitle = "Impact of a 0.1 increase in negativity on the popularity of 
    ministers and shadow ministers tweets") 

![](/Users/piers/Documents/GitHub/piersyork.github.io/_posts/twitter_analysis_files/figure-markdown_strict/popularity_model-1.png)
Perhaps this says more about Labour’s twitter followers than it does
about the effectiveness of negativity for the Labour party’s popularity.
<br/><br/><br/><br/>
