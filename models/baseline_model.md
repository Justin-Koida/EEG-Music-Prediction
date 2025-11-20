# Data and Features

I explored initial models with the provided cleaened EEG data. The structured of the cleaned EEG is as follows. There are 10 cleaned EEG files, each file corresponding to one song. Each file is a .mat file in the form (125, time, 20). This translates to 125 channels, t amount of time the data is recorded over, and 20 participants. The files are named in format song#_Imputed.mat where # is the song id. The creators of this dataset used a unique 'trigger' to song mapping which is why the song names are in this format.

In this initial model, I wanted to create a solid baseline for which I could compare future models. Since I used the data cleaned by the researchers of this dataset, this provides data for solid baseline which I can compare to future models created from this data, and also future models where I apply my own cleaning techniques. 

For this model, I computed bandpower across different frequencies for each channel in every song for a given participant. For EEG, this is a standard feature that is calculated and great for baseline models. I calculated power based off the following Band frequency mapping:

BANDS = {
    "delta": (1, 4),
    "theta": (4, 8),
    "alpha": (8, 12),
    "beta": (12, 30),
    "gamma_low": (30, 45),
}

The ranges above are just standard frequency ranges for brain waves. 

Bandpower measures the total energy an EEG signal contains within a specific frequency range (delta, theta, etc.). I computed this using Power Spectral Density (PSD) estimate derived from Weltch's method. Bandpower is then calculated by integrating the PSD using upper and lower bounds according to frequency ranges (called 'BANDS' in my code).

It is important to note that for each song and participant I computed power across the full time and did not chunk into windows. 

# Initial Model

For my initial model, I tested a simiple logistic regression baseline. I converted user's enjoyment score to a binary target representing not liked or liked. I defined liked as any enjoyment_score greater than or equal to 6, and not liked as anything below 6 (enjoyment score is a discrete scale from 1 to 9).

Features in this model included only the power features calculated per channel. With 10 songs for each 20 participants, there are 10 x 20 = 200 observations in my dataset. With 125 channels across 5 frequency ranges, there are 125 x 5 = 625 features in my dataset.

The features are all numeric data, so I applied a standard scalar in order to ensure all features are on the same scale. I applied cross validation in order to ensure robust evaluation metrics since I have an extremely small dataset. I used a Leave One Group Out cross validation technique in order to ensure no leakage when evaluating (having a standard split would mean the model gets trained on a participant and may memorize participant specific patters). In this technique, I had 20 splits each had a participant left out and used as the test set.

In addition to my first logistic regression, I created 2 other logistic regression where I experimented with the ground truth for the target variable. In one model I made the like class for the enjoyment_score >= 5, for another model I made the like class for enjoyment_score >= 7.

Additionally, I created a linear SVM that used ground truth of liked for enjoyment_score >=6. Of all these models, the first logistic regression baseline seemed to perform the best.

# Model Evaluation

For evaluation I computed accuracy, precision, and recall for each fold. I computed the mean of each metric across all folds. My baseline logistic regression (with enjoyment_score >= 6 as liked and < 6 as not liked), I achieved a 62.5% accuracy for my best baseline logistic regression. Additionally, I computed precision and recall on the whole data itself (Pooled Across Participants) in addition to the per fold average of precision and recall. This is because for some folds, one class is not defined. By computing precision and recall pooled across the whole dataset from each fold, I ignore the case where the class is missing.

**Best Baseline Logistic Regression**

![alt text](<Screenshot 2025-11-19 at 10.02.14 PM.png>)

*Pooled across partcipants (not mean of folds) for baseline logistic regression with target variabed defined as enjoyment_score >= 6 as class 1 (liked) and enjoyment_score <6 as class 0 (not liked)*

**Strengths and weaknesses of the baseline**

This baseline is a solid starting point. It performs better than random from a simplistic power feature calculation. This baseline allows for easy comparison to how future feature engineering impacts model performance compared to a simplistic power feature based model. 

The weakness of this model is that it has mediocre performance as evaluated by accuracy, precision, and recall. Addionally, since badpower features were aggregated over the entire song, the model cannot capture temporal dynamics like short-term burst in specific frequencies. A weakness overall is that this dataset is extremely small. Any models trained will have this weakness though.

**Possible reasons for errors or bias**
Since this is a very small dataset with class imalance, perhaps this could cause errors and bias in my analyisis. Additionally, simple version of features were created, more in-depth features will be created, however simple features may not fully be captuing the complexity within the data.

# Ideas from Initial Modeling/Next Goals

Based on the performance of my initial baseline model, there are several directions that I plan to explore to improve prediction accuracy and more deeply capture the neural dynamics present in the EEG data.

**Future feature exploration related to bandpower will include:**

- sliding window bandpower: an approach to capture temporal components of bandpower by computing power in short windows (windows being time frames e.g. 2 second window).

- relative bandpower: this is just power in a band normalize by the total power

- log bandpower: taking the log of bandpower

**Additionally, future exploration may incoperate the use of scalograms and spectograms.**
Scalograms are 2 dimensional time frequency representations derived from the Continous Wavelet Transform (CWT)

Spectograms are 2 dimensional time frequecy representations derived from the Fast Fourier Transform (FFT)

Both these methods preserve more information than bandpower typically does which will allow for the model to capture more meaningful patterns. Since these can be represented as images, a CNN can be trained on the generated scalograms and spectograms.


**Target Variable Ideas**

Some additional ideas are changing the target variable. Currently I have the target variable as a binary not liked/liked. I could tinker with the amount of classes I have for the target feature. I could create a model that uses the current scale of 1 to 9, and try to predict across 9 classes. Another idea would be to create a disliked, neutral, and liked classes. For this, I would use 1-3 as dislked, 4-6 as neutral and 7-9 as liked. 

**Tuning**

Did not apply hyperparameter tuning, this will be important to incoperate for future more in-depth models.

# Long-term Future Directions

Currently I use the cleaned EEG data provided by the creators of this dataset. I would like to clean the raw EEG file myself in order to incorperate some ideas that I have. 

1) Currently the cleaned EEG data is downsampled from 1000 hz to 125 hz. I would like to instead downsample to 256 hz. The primary reason for this is because I would like to eventually be able to predict upon users wearing a Muse 2 Headband. This device reads EEG data from 4 channels and samples at 256 hz (what I want to downsample to). I would need to somehow project the 125 channels used in this dataset to the 4 channels the Muse 2 Headband utilizes.

2) Validate the cleaning results of the researchers who created this dataset.

If I were able to project the 125 channels in this dataset to the 4 channels of the Muse 2 headband, I could validate model training on EEG data that I collect (I have access to a Muse 2 Headband).