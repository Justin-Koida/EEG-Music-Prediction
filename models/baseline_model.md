# Data and Features

I explored initial models with the provided cleaened EEG data. The structured of the cleaned EEG is as follows. There are 10 cleaned EEG files, each file corresponding to one song. Each file is a .mat file in the form (125, time, 20). This translates to 125 channels, t amount of time the data is recorded over, and 20 participants. The files are named in format song#_Imputed.mat where # is the song id. The creators of this dataset used a unique 'trigger' to song mapping which is why the song names are in this format.

In this initial model, I wanted to create a solid baseline for which I could compare future models. Since I used the data cleaned by the researchers of this dataset, this provides data for solid baseline which I can compare to future models created from this data, and also future models where I apply my own cleaning techniques. 

For this model, I computed power across different frequencies for each channel in every song for a given participant. For EEG, this is a standard feature that is calculated and great for baseline models. I calculated power based off the following Band frequency mapping:

BANDS = {
    "delta": (1, 4),
    "theta": (4, 8),
    "alpha": (8, 12),
    "beta": (12, 30),
    "gamma_low": (30, 45),
}

It is important to note that for each song and participant I computed power across the full time and did not chunk into windows. 

# Initial Model

For my initial model, I tested a simiple logistic regression baseline. I converted user's enjoyment score to a binary target representing not liked or liked. I defined liked as any score greater than or equal to 6, and not liked as anything below 6 (enjoyment score is a discrete scale from 1 to 9).

Features in this model included only the power features calculated per channel. With 10 songs for each 20 participants, there are 10 x 20 = 200 observations in my dataset. With 125 channels across 5 frequency ranges, there are 125 x 5 = 625 features in my dataset.

The features are all numeric data, so I applied a standard scalar in order to ensure all features are on the same scale. I applied cross validation in order to ensure robust evaluation metrics since I have an extremely small dataset. I used a Leave One Group Out cross validation technique in order to ensure no leakage when evaluating (having a standard split would mean the model gets trained on a participant and may memorize participant specific patters). In this technique, I had 20 splits each had a participant left out and used as the test set.


# Model Evaluation

For evaluation I computed accuracy, precision, and recall for each fold. I computed the mean of each metric across all folds. My baseline logistic regression (with enjoyment >= 6 as liked and < 6 as disliked), I achieved a 62.5% accuracy, 


# Ideas from Initial Modeling/Next Goals

Based on my initial model, I have a few ideas of what I want to explore next. First, I could change up the power features in my model. There are techniques for capturing power within a specific time frame. This could lead to more accurate results.

Addionally, future models could incoperate the use of scalograms and spectograms. 

Scalograms are _

Spectograms are_


Some addional ideas are changing the target variable. Currently I have the target variable as a binary not liked/liked. I could tinker with the amount of classes I have for the target feature. I could create a model that uses the current scale of 1 to 9, and try to predict across 9 classes. Another idea would be to create a disliked, neutral, and liked classes. For this, I would use 1-3 as dislked, 4-6 as neutral and 7-9 as liked. 


This baseline is a solid starting point. It performs better than random from a simplistic power feature calculation. This baseline allows for easy comparison to how future feature engineering impacts model performance compared to a simplistic power feature based model. 

The weakness of this model is that it has mediocre performance as evaluated by accuracy, precision, and recall. A weakness overall is that this dataset is extremely small. Any models trained will have this weakness though.

# Future Directions

Currently I use the cleaned EEG data provided by the creators of this dataset. I would like to clean the raw EEG file myself in order to incorperate some ideas that I have. 

1) Currently the cleaned EEG data is downsampled from 1000 hz to 125 hz. I would like to instead downsample to 256 hz. The primary reason for this is because I would like to eventually be able to predict upon users wearing a Muse 2 Headband. This device reads EEG data from 4 channels and samples at 256 hz (what I want to downsample to). I would need to somehow project the 125 channels used in this dataset to the 4 channels the Muse 2 Headband utilizes.

2) Validate the cleaning results of the researchers who created this dataset.

If I were able to project the 125 channels in this dataset to the 4 channels of the Muse 2 headband, I could validate model training on EEG data that I collect (I have access to a Muse 2 Headband).