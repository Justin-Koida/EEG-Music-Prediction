# EEG-Music-Prediction
Author: Justin Koida

## Abstract
This paper explores whether EEG (electroencephalogram) data can be used to predict an individual's enjoyment of a given song. Models tested in this project were derived from features extracted from EEG data of participants in a study. Utilizing bandpower features and generation of scalograms, this project examines multiple approaches to modeling EEG data. Results from bandpower and scalogram features performed relatively similarly with highest performing accuracy being 62.5% for bandpower-based models and 62% for scalogram-based models.   

## Introduction
Understanding how an individual responds to music has been a topic of interest in neuroscience and psychology. Music is a complex stimulus that engages emotional and cognitive stimulus in the brain. The current knowledge of how the brain works is limited. This project explores how EEG data can be used to determine an individual's enjoyment of a song. EEG data can be thought as real time measurements of the brain's electrical activity. This type of data is relatively cheap compared to other collection methods of brain data, and is becoming highly accessible to consumers. By using EEG data we can observe how the brain responds to music stimuli. 

The ability to determine a listener's enjoyment of a song can largely impact the current knowledge of the brain. By extracting frequency domain features and applying supervised machine learning models, I evaluate the strength of the relationship between neural activity represented through EEG and subjective music enjoyment. This project offers contribution to the larger effort of integrating neuroscience and machine learning.

## Data 
The Data utilized in this project was taken from the Naturalistic Music EEG Dataset – Tempo (NMED-T). In this study, Stanford researchers conducted a multifaceted experiment measuring responses to natural music listening. There were 20 participants in this study, and each listened to 10 specific songs as shown in table xxx. Participants were tasked with listening to these songs while researchers collected EEG data through 128 electrodes placed on targeted areas around the brain. 
More information about how the study was set up can be found in the citation section.

Researchers provided a cleaned dataset in the form of 10 .mat files, raw EEG files for each participant, participant information, and participant ratings. In this project I utilized the cleaned EEG data. The structure of the cleaned EEG is as follows. There are 10 cleaned EEG files, each file corresponding to one song. Each file is a .mat file in the form (125, time, 20). This translates to 125 channels, the amount of time the data is recorded over, and 20 participants. The files are named in format song#_Imputed.mat where # is the song id. The creators of this dataset used a unique 'trigger' to song mapping which is why the song names are in this format. Note that each channel maps to an electrode used to gather data; it is also important to note that the amount of channels does not match to the amount of electrodes due to preprocessing applied by the researchers. 

## Feature exploration

The models created in this project were created from two primary features. First, bandpower features were calculated in order to train bandpower-based models. Second, scalograms were generated in order to train scalogram-based models. 

Bandpower is a feature based on frequency that quantifies how much energy an EEG data series contains within specific bands of frequency. In this project, standard frequency bands were chosen.

BANDS = {
    "delta": (1, 4),
    "theta": (4, 8),
    "alpha": (8, 12),
    "beta": (12, 30),
    "gamma_low": (30, 45),
}

These bands were utilized since specific frequency ranges correspond to specific mental states. For example, the alpha frequencies usually correspond to something like relaxation and disengagement.

Bandpower is calculated through a two step process. First, the power spectral density (PSD) of the EEG signal is estimated. This was done by using Welch’s method, which essentially splits signals into overlapping segments, computes the Fast Fourier Transform (FFT) on the segment, then averages the results to produce a smooth estimate. This is done by importing scipy.signal’s welch function. Second, the PSD is then integrated, utilizing a function called calculate_bandpower, across each target band as defined as BANDS in my code. This is done for each song for each participant across the 125 channels resulting in a 200 x 625 dataframe (ROWS: 20 participants x 10 songs — COLS: 5 Bands x 125 Channels).

Scalograms are a time frequency representation using the Continuous Wavelet Transform (CWT). While bandpower specifically looks at total energy of specific frequencies within a time frame, scalograms aim to preserve how frequency changes over time. The scalograms generated are represented as images with dimensions being 64 x 64.

Scalograms are computed by applying the CWT to the EEG data. The CWT works by applying an oscillatory wavelet across the EEG signal at various time positions and scales with each scale corresponding to a specific frequency range. This allows the transform to capture both low and high frequency patterns. It is also suitable for capturing long and short duration patterns. This project utilizes the Morlet wavelet as the chosen wavelet in the CWT and implemented using PyWavelets library. 

A function called compute_scalogram was developed in order to generate the scalogram image. The function takes as input a single EEG channel, the sampling frequency, the target frequency range, the number of frequencies to compute, downsampling parameters, and output image size. It produces scalograms that are normalized and log scaled. Essentially this function is utilized to create many scalograms across multiple channels, songs, and participants. These scalograms are then used to train a CNN classifier which were then used to train a CNN classifier for the binary task of predicting whether a participant liked a given song.


## Models
All models created were evaluated using the same metrics in order to make them directly comparable. Additionally, models all utilized cross validation, specifically in the form of Leave-One-Group-Out Cross-Validation (LOGO). In the context of this project, LOGO works by withholding a singular participant (1 of 20) from training split, and evaluates the model on the singular held out participant. The LOGO cross validation utilized 20 splits, so every participant had a chance to be evaluated in the test set. For model training, it is important to ensure there are no leakages because it can inflate performance and create misleading metrics. LOGO mitigates leakage because it ensures the model is never trained then evaluated on a person it has seen; instead, the model is evaluated on an individual it has never seen before.

### Random

This is the model used for comparing from baseline. This is a random model meaning the model randomly guesses for its prediction. The accuracy for this model was 0.46499999999999997. A confusion matrix of predictions is shown below.

![alt text](<Screenshot 2025-12-04 at 11.23.12 PM.png>)

### BandPower-Based

Multiple power-based models were created using features derived from absolute bandpower, total bandpower of a specific band (as defined in BANDS) for a given channel. A bandpower baseline using log absolute bandpower, which represents the log of the total power within a targeted frequency band, was first created. Then, models were created using the combination of absolute bandpower, log absolute bandpower, and relative bandpower. Relative bandpower normalizes each band’s absolute bandpower by using total bandpower across all bands for a given channel. This produces a ratio representation of how energy is distributed across frequency bands.

The baseline models include Logistic Regression and Support Vector Machine.

The models trained that utilize all bandpower features are Logistic Regression, Support Vector Machine, and Random Forest.

No hyperparameter tuning was conducted besides manually tuning parameters due to time consumption.


### Scalogram-Based

Multiple scalogram-based models were created using scalograms generated with slightly different parameters. Parameters that were tuned include:

max_seconds: controls how many seconds of the song are used in creating the scalogram. This parameter is used when selecting the data to feed into the compute_scalogram function.

k: top k most impactful EEG channels in the bandpower base model that are then used to create the scalograms (mostly just use k = 8)

target_fs: changes how much downsampling occurs.

A custom CNN was created in order to train on the scalograms. This CNN is designed to extract time frequency patterns from the 64 x 64 scalogram images. It consists of three convolutional blocks, which each contain a convolution layer, batch normalization, ReLU activation, and a 2x2 max pooling operation. After the three convolution blocks, global average pooling compresses the feature into a single value. A dropout layer is also applied for regularization. The output is a single output for binary classification. Note that ReLU activations are applied in the forward pass function.

Additionally, a pretrained ResNet18 model was initially tested, though did not yield improved results and increased time complexity, so it was not further pursued. 


### Results

#### Best Models vs Random
![alt text](<Screenshot 2025-12-04 at 10.54.28 PM.png>)

#### Bandpower-Based Models Comparison
![alt text](<Screenshot 2025-12-04 at 10.55.19 PM.png>)

Results were not the best, but an improvement from the random baseline. Both the scalogram-based CNN models and bandpower-based models performed about .15 higher in terms of accuracy compared to the random baseline. 

The best model for the bandpower-based models was the Logistic Regression of the log of the absolute bandpower. This was the baseline bandpower model which was surprising. This model can be viewed under models/power_baseline.ipynb.

The best model for the scalograms was the model that was trained on scalograms generated from the full song, utilizing the top 8 channels derived from the best bandpower model. To view the parameters used for creating the scalograms in this model, the call to compute_scalogram is shown below:

img = compute_scalogram(
                sig,
                fs,
                fmin=1.0,
                fmax=45.0,
                num_freqs=32,
                out_size=(H, W), <- Where H = 64 and W = 64
                target_fs=64,
            )

## Limitations

The dataset used in this project only contained 20 participants. The dataset was very small which makes it difficult to train a high performing model.

Training these models takes time. This project was limited to exploring only specific areas due to limited computational power (computer crashed multiple times).

## Future Works

Currently I use the cleaned EEG data provided by the creators of this dataset. I would like to clean the raw EEG file myself in order to incorporate some ideas that I have. 

1) Currently the cleaned EEG data is downsampled from 1000 hz to 125 hz. I would like to instead downsample to 256 hz. The primary reason for this is because I would like to eventually be able to predict upon users wearing a Muse 2 Headband. This device reads EEG data from 4 channels and samples at 256 hz (what I want to downsample to). I would need to somehow project the 125 channels used in this dataset to the 4 channels the Muse 2 Headband utilizes.

2) Validate the cleaning results of the researchers who created this dataset.

If I were able to project the 125 channels in this dataset to the 4 channels of the Muse 2 headband, I could validate model training on EEG data that I collect (I have access to a Muse 2 Headband).

3) Exploring windowing techniques could improve model performance. However windowing techniques are very computational expensive and was not within the scope of this project.

4) Collecting more data because in general, the more data, the better.

5) Create spectrograms and train off those.

6) Ensemble across various models. For example, currently I could ensemble the Scalogram-based CNN with the Logistic Regression Bandpower model.

7) Hyperparameter tuning -- did not do because of computational cost.


## Conclusion

The results of this project were mediocre. All models failed to reach high accuracy scores, and balanced high precision/recall scores. While many models were tested across various methodologies, EEG-based signals were unable to be identified to predict music enjoyment. This could be due to not applying more powerful techniques such as the sliding window approach; though this was due to lack of computational power. Even though models failed to reach high accuracy scores, the models created were able to capture a small signal, as accuracy was about .15 higher than the random baseline model.

## Citation for Data

Steven Losorelli, Duc T. Nguyen, Jacek P. Dmochowski, and Blair Kaneshiro. 2017. NMED-T: A Tempo-Focused Dataset of Cortical and Behavioral Responses to Naturalistic Music. In Proceedings of the 18th International Society for Music Information Retrieval Conference, Suzhou, China.
https://ismir2017.smcnus.org/wp-content/uploads/2017/10/198_Paper.pdf