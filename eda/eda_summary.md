# Goal
Using Electroencephalography (EEG) data of a user listening to a song, predict if the user likes the song or dislikes the song. EEG data is simply a method of recording eletrical activity in the brain which is captured by placing electrodes on the scalp. EEG data is a time series data representing brain activity, recorded across multiple channels. 

# Dataset

The dataset I chose is called Naturalistic Music EEG Dataset - Tempo (NMED-T) and can be found here -> https://purl.stanford.edu/jn859kj8079. I chose this dataset for a few major reasons. First, this dataset contains a like/dislike scale in order to determine if a subject likes a song or not. Second, this dataset contains the actual songs the subjects listened to; a future direction of this project is to recommend songs based on songs liked classified from EEG data. In order to do this, knowing what songs a subject likes is crucial since these songs have various features which can be pulled from Spotify API. Third, this dataset contains EEG data from subjects listening to a given song. I am hoping to use this EEG data for classifiying if the subject likes the song or dislikes the Song. 

This dataset comes with data that is already cleaned (even the EEG data), so no code was creating for cleaning the data. However, I may try to preprocess the raw data in subsequent iterations. Some preliminary code for loading in the preprocessed EEG and the Raw EEG were given in the code.

# EDA

The data of this dataset is structured with rating information, demographic information, and files for each subject's EEG data. My eda begain with exploring the rating information. The rating system was split into two parts for this dataset: familiarity and enjoyment. For the purpose of this project, I only care about enjoyment since I am predicting liked/disliked. Thus I explored the enjoyment data. What I found was that the average enjoyment per song across all subjects usually lie between a 5 to 7 (on a 1-9 scale where 1 is dislike and 9 is really liked); songs 3 and 10 had lower average score, so these two songs were not really liked by the participants. 

![alt text](<Screenshot 2025-11-13 at 12.15.42 AM.png>)


Most ratings for the songs are clustered in the 5 to 7 range. 

![alt text](<Screenshot 2025-11-13 at 12.29.55 AM.png>)

I plotting the rating distribution for each song. 

Notably song distributions include:

song 6 which had lots of high ratings

![alt text](<Screenshot 2025-11-13 at 12.31.56 AM.png>)

song 10 which had lots of low ratings

![alt text](<Screenshot 2025-11-13 at 12.31.25 AM.png>)

Next, I explored some of the subject metadata. Of the relevant features, I looked at their distribution. What I found was that for all these variabes: age, nYearsTraining, and weeklyListening were all skewed right. These variables may be features that could be added into the model, however most of the features will be derived from the EEG data.

![alt text](<Screenshot 2025-11-13 at 12.33.46 AM.png>)

Addionally, from this eda, I learned that I am not missing rating information, and I gained an understanding of the rating distributions for each song. I learned information about demographic info which could potentially be incorperated into the final model.

Note: I have not explored the EEG data part though I will in the future. EEG feature extraction will be the majority of work in this project; I have started to look into how features can be extracted from this type of data. Future updates will be made.

# Issues

There are severe limitations to the amount of data I am able to get. I've looked through multiple datasets, and all datasets are relatively small. Many of the datasets did not have all the information I wanted for this project (such as song listened to, or rating scale) so I could not use those datasets. Perhaps combining data from multiple sources would allow for me to easily test my model with a comfortable amount of datapoints.

Loading in the data for this was also difficult since the data is structured a specific way and is in .mat format. 

Addionally, there seemed to be few datapoints for highly disliked songs (rating = 1 and 2) and highly liked songs (rating = 9). This could pose a problem for training a robust model to predict highly like/dislike.

# Dataset Citation

Steven Losorelli, Duc T. Nguyen, Jacek P. Dmochowski, and Blair Kaneshiro (2017). Naturalistic Music EEG Dataset - Tempo (NMED-T). Stanford Digital Repository. Available at: http://purl.stanford.edu/jn859kj8079