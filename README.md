**1.	Introduction**

This project aimed to develop a machine learning model capable of predicting emotional states, specifically valence and arousal, from audio files. The process involved converting audio data into MIDI format, extracting relevant features, and building a predictive model.

**2.	Dataset and Data Preparation**

The dataset used for this project is the Database for Emotional Analysis of Music (DEAM) dataset, which includes a collection of audio files with corresponding annotations for their emotional attributes. The audio files were converted to MIDI files using the Basic Pitch library.

**2.1.	Feature Extraction**

The next step was to extract meaningful features from the MIDI files. The following features were selected to capture various aspects of the musical content, which are relevant to predicting emotional states:
•	Pitch: The pitch of each note was extracted to analyze the melodic content.
•	Duration: The duration of each note was measured to understand rhythmic patterns.
•	Velocity: The velocity of each note was recorded.
•	Tempo: The overall tempo of the piece was noted as it impacts the overall mood.
•	Pitch Class Histogram: This feature captures the distribution of pitch classes.
•	Note Density: This metric indicates how many notes are played per unit time, reflecting the piece's complexity.
•	Melodic Intervals: Differences between successive pitches, which helps in analyzing melodic structure.

**3.	Data Issues and Presented Solutions**

The dataset included missing values, particularly in time-series features where some MIDI files did not contain continuous sequences of notes due to the varying length of their corresponding original audio files. To handle this, custom masking layers were introduced in the model to manage these missing values effectively. Further on, the custom masking layers appeared to cause several issues when saving/importing the model. 
A revised approach consisted of filling NA values with 0s and then using a generic Masking layer on the 0 values. 

**4.	Model Architecture and Issue Resolution**

Initially, a CNN was selected. CNNs are typically effective for spatial data and have shown great success in image processing. However, upon further research I learned that RNNs are generally more suitable for time-series or sequence data such as MIDI features due to their ability to handle temporal dependencies and sequence patterns.
Realizing the limitations of CNNs for this problem, the decision was made to transition to RNNs. 

**4.1.	First Model**

The first RNN model was complex and incorporated a much larger number of features to capture a wide range of temporal dependencies and sequence patterns in the MIDI 
data based on the provided training data annotations. This model was built to leverage the full range of extracted features, providing a comprehensive approach to learning the relationships between musical features and emotional attributes.

**4.1.1.	Layers**

RNN Layers: The model incorporated two SimpleRNN layers with 64 units each.
Masking Layer: Implemented to handle varying sequence lengths by ignoring padded zero values, focusing the model's learning on meaningful time steps.
Dense Output Layer: A final dense layer was used to predict valence mean, valence std, arousal mean, and arousal std.

**4.1.2.	Issues**

**Zero Loss**

One notable issue was the model consistently reporting a loss of zero, which indicated a problem with the loss function or model configuration. By addressing the input shape, feature representation and data/model compatibility issues, the model’s loss function was recalibrated, and performance metrics were more accurately reported.

**Input Shape Issue**

As MIDI features were extracted from the audio files and added to the training dataset, the model was expecting a very large input shape. This, however, was not the intended input shape of the final model as per the required use-case, as the model expected the training data columns as part of its input shape, yet the goal was to develop a model that predicts valence and arousal based on only the MIDI files’ extracted features. Further explanation of how this issue was addressed can be found in the below section.

**4.2.	Second Model**

A second, more simplified model was developed to accommodate for the intended purpose of the project. This model’s input shape was much smaller (22) in comparison to the more complex initial model mostly due to the time-series columns in the training dataset. This model was streamlined to focus on essential features, reducing complexity while retaining predictive power. It aimed to address issues encountered with the initial model’s input shape and performance.

**4.2.1.	Layers**

This model used a Dense Neural Network instead of an RNN due to its direct approach to handling fixed-size input features. Dense NNs are well-suited for non-sequential data, making them ideal for predictions based on extracted MIDI features.
Dense Layers: Two dense layers with 32 and 16 neurons respectively were employed, using ReLU activation to extract meaningful patterns from the input features.
Custom Activation in Output Layer: A custom activation function ensured non-negative predictions by setting a lower bound of 0.37, matching the minimum values of the training data. This adjustment prevented negative outputs and improved prediction accuracy.

**4.2.2.	Issues**

**Negative Predictions**

Upon evaluating the model, a review of the model’s architecture revealed that the output layer was not using a selected activation function. It was configured to use ReLU, but that heavily impacted the model’s performance. To address this issue, this was done by developing a custom activation function to the output layer to enforce non-negativity. This custom function used the K.maximum(x, 0.37) method, which ensures that the minimum value returned by the model is 0.37 (the minimum value of the test data). This adjustment helped avoid any output below this threshold, in turn addressing the problem of negative predictions.

**4.3.	Knowledge Transfer**

The knowledge transfer between models involved understanding the limitations of the initial complex model and applying this understanding to the design of the simpler model.
A feature extractor model was used by extracting the output of the second SimpleRNN layer. This was then used to transfer the knowledge from the initial to the second, more simplified model.

**5.	Model Output**

Each output from the model corresponds to specific attributes:
•	Valence Mean: Indicates the average positivity or negativity of the music.
•	Valence Std: Measures variation in emotional tone.
•	Arousal Mean: Reflects the average energy level.
•	Arousal Std: Shows variation in energy levels.

**6.	Results**

Model 1 (RNN-based): Demonstrated strong capabilities in capturing temporal dependencies, achieving a loss of 0.4437 on the validation set. It effectively handled mid-range valence and arousal values but struggled with extreme values.
Model 2 (Dense NN): Achieved a validation MSE of around 1.057. This simpler model excelled in scenarios requiring quick and efficient predictions based on static features, although it was less adept at modeling complex temporal patterns compared to the RNN model.
