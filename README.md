# NBA_Predictions
By Ryan Ahmed, Ajay Ambati, Sathya Balakumar, Hamza Shaikh, and Vivek Mahapatra

Introduction
The National Basketball Association (NBA) is composed of some of the world’s top notch athletes. These athletes are placed on one of 30 NBA teams to compete for a championship. In order to get there, however, teams must win games during the regular season, which is composed of 82 games. Typically, if a team wins more than half its games (42 or more) in the regular season, there is a good chance it will make the playoffs. There are several factors to winning more than 50% of games and securing an above 0.500 record. These include shooting percentages, injuries, opponent field goal percentages, points per game, and many more advanced statistics. Our goal in this project is to analyze the various features that influence winning games and predict whether an NBA team will have an above 0.500 record, or at least 43 wins. We will rely on advanced NBA statistics from previous NBA seasons. This is a binary classification problem in which a ‘1’ signifies that the team will have an above 0.500 record, while ‘0’ signifies otherwise.

Datasets
We will be focusing our project on the following NBA seasons: 2019–20, 2020–21, and 2021–22. We will be training using the data from the 2019–20 and 2020–21 seasons, and we will predict whether a team will have an above 0.500 record on the 2021–22 season. The data for these 3 years was collected from the following website: https://www.basketball-reference.com/. Each dataset has 30 rows to represent each of the 30 NBA teams. There are also 50 columns, which consist of both offensive and defensive stats such as points, rebounds, assists, steals, and blocks. Here the links to where we obtained the data for the three NBA seasons from 2019 to 2022:

https://www.basketball-reference.com/leagues/NBA_2020.html

https://www.basketball-reference.com/leagues/NBA_2021.html

https://www.basketball-reference.com/leagues/NBA_2022.html

Data Exploration and Feature Engineering
Before we describe our approach to exploring and analyzing the data, we describe the meanings of each of the features in the table below. Note that the features of the datasets are divided into two sections: team and opponent stats. We have indicated the team stats and opponent stats by attaching “_OFF” and “_DEF”, respectively, to the feature names because how well a team performs on offense or defense may indicate whether they won most of their games. For example, “FG_OFF” and “FG_DEF” both mean the number of field goals, but the former means on the team side (on offense) while the latter means on the opposing side (on defense).



In the exploratory analysis phase, we begin our analysis by first using an untuned gradient-boosting model (Catboost) to see how the model performs on the given data without any modifications. Because Catboost is a robust model, it unsurprisingly performed well on the data. On the training data, the model achieved perfect accuracy, precision, and recall, which indicates that the model has sufficiently learned its training set. And on the testing data, the model achieves an accuracy of 83%, precision of 93%, and recall of 78%, which is pretty good.

We extracted the top features used for training the model and displayed the results below. One thing to note is that the top three features (PTS_DEF, 3P%_DEF, and FG%_DEF) are all defense features. Since they are at least twice as important as any other feature, the model suggests that the team that performs well on defense is more likely to win their games, which makes sense.


Now that we’ve modeled the data without modifications and analyzed the feature importances, we’ll try to analyze the data and modify it so that our Catboost model performs better. We first performed some basic preprocessing like checking the missing values, outliers, feature data types, and feature correlations. Running the describe method and plotting the histograms of each feature, we verified no missing values and normally distributed features, indicating well-behaved features.

However, we noticed that all the features are numeric except one: the team name feature. Since the team name feature is full of many unique strings, models may struggle to use it during training. Thus, we need to modify this feature so that it’s more meaningful to the model under training. We noticed how the teams are divided into Eastern and Western Conferences according to the NBA season summary. Therefore, we replaced the team column with the Eastern Conference feature and mapped the teams accordingly (1 for Eastern Conference, 0 for Western Conference). We created the “transformed teams” dataset based on this change and proceeded with the other preprocessing techniques.

When creating the correlation matrix of the features, there were many features that naturally showed high correlation. For example, the features FG_DEF and FG%_DEF had a high correlation because FG%_DEF is a function of FG_DEF. These highly correlated features begged the question: if FG%_DEF represented the same information as FG_DEF and FGA_DEF combined, then why did we need both representations? We created two datasets to test if a model would perform better on one type of data or the other. The “ratios only” dataset contained only the ratios (e.g., FG%_DEF), while the “counts only” dataset contained only the counts (e.g., FG_DEF and FGA_DEF). We then continued analyzing the correlations between the features.

When analyzing the correlations more, we noticed something unexpected. Since features like FG_DEF and FG_OFF represented similar information, we expected them to be highly correlated. Instead, these similar feature pairs actually had some of the lowest correlations in the matrix. This observation may suggest that OFF and DEF features are significantly different, so the split in the features should be kept. However, to test this idea, we created datasets where the OFF-DEF pairs are averaged. The “averaged OFF/DEF features” datasets reflect this change. Similarly, we created datasets for the “ratios only” and the “counts only” using the averaged data as the source. The table below contains the feature engineered datasets and their descriptions to summarize.


After creating all these datasets, we train a basic Catboost model to all these datasets, evaluate their performance on the test data, and extract their most important features in training. The table below shows the performance results, and the figures below represent the feature importance.

Although none of the models performed better than the original data, the “transformed teams” and “ratios only” datasets were on par. The results for “transformed teams” were unsurprising because we only created and deleted one feature, and the model didn’t really use the new feature, resulting in similar behavior to the original data. However, the results for “ratios only” were fascinating because we removed many count features and obtained the same performance. This suggests that the ratio features hold more weight than their counterparts when training a model. This claim is supported by the feature importances of the original data and the ratios only data, where the top features of both were ratio features.

Naturally, the results for the “counts only” dataset were worse than the “ratios only” dataset. We have seen that ratio features give more information about the target variable to the model. For example, the feature FG%_DEF conveys the information of FG_DEF and FGA_DEF and that they are related by division. However, providing only FG_DEF and FGA_DEF loses the relationship between them. Consequently, the model does not see the relationship and treats them individually, losing meaningful information.

When we performed the correlation analysis, we saw that it was meaningful to keep the OFF and DEF features separate. Consequently, averaging the features resulted in poor results as shown in the table, which we expected. However, the “averaged features with counts only” dataset performed better than expected, achieving better precision but worse accuracy and recall than the normal “counts only” dataset. Whereas all the other datasets clearly favored the DEF features, these datasets had a mix of top features. Furthermore, the feature PTS, which partially corresponds to PTS_DEF, is no longer a strong training feature.

Needless to say, the averaged datasets won’t be used to tune models. However, “transformed names”, “ratios only”, and “counts only” are strong datasets for tuning, so we passed these sets to the next phase.








Conclusion
By testing NBA data against multiple classifiers, we were able to predict the expected number of teams (among 30 NBA teams) that would win 50% of their games in the 2021–2022 NBA season. The results were as follows:


According to the data above, Logistic Regression did the best among the classifiers.

The feature engineering solution created by one of our team members affected only one of our classifier’s accuracy; however, the accuracy increased by a significant 3% (RandomForest before tuning changed from 74% to 77%). Thus, it can be concluded that the benefits of feature engineering, though effective in some cases, are minuscule compared to most of the boosted classifiers we tested. A common consensus in the NBA is that a good defense wins the game. This correlation is evident in our data, as there exists a clear relationship between our PTS_DEF category (the category that labels the defensive points per team) receiving the highest correlation score of 20 amongst other columns. As the 2019–2020 year was affected due to the COVID-19 pandemic, some of the resulting predictions could be skewed in favor of some teams over others. One possible change that could aid this problem would be to add more gameplay years to the dataset, as a higher volume of input data would lessen the effect that the outlier of 2019–2020 causes.

Link to slides (Presentation): https://docs.google.com/presentation/d/1eoGdEp8QWJMOM4vIEJRd1_RYDZkQs_TM9h2SU1VhDLo/edit?usp=sharing
