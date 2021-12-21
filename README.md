# Human Acitivity Recognition Using Labeled Smartphone Data

### ***Project by Jonny Hofmeister***

The data for this project comes from the [UCI Machine Learning Repository](https://archive.ics.uci.edu/ml/datasets/Smartphone-Based+Recognition+of+Human+Activities+and+Postural+Transitions) and was collected by researchers at the University of Genova. [This paper](https://www.esann.org/sites/default/files/proceedings/legacy/es2013-84.pdf) describes their data collection and construction process and [this video](https://www.youtube.com/watch?v=XOEN9W05_4A) shows an exaple of a data collection trial.

## Business Understanding

The goal of this classification model is to use smartphone-based sensor data/information to determine what kind of activity the user is performing. There are 6 activities labeled and recorded in this data, three are static postures (Sitting, Standing, and Laying Down) and three dynamic activities (Walking, Walking Upstairs, and Walking Downstairs). The researchers have also labeled and included the transitions between these activities, but intial modeling attempts in this project will seek to classify just the initial 6 activities.

Understanding the activities of persons can have large benefits towards understanding their health and daily patterns. Recognizing and recording these patterns can help determine how activites affect health, sleep, etc. For example, tracking time sitting down can help researches correlate time spent sitting to negative health effects and them begin to inform patients on how to change their activity patterns to improve health.

Smartphones are an effective way to collect users activity data for multiple reasons. First, smartphones already contain all the necessary sensors to record motion data. Second, they are accessible as most people already have them. Third, using them to track motion is non-invasive medically and in terms of how they live their life - data can be collected just by turning on an app and not changing any other daily patterns.

The researchers who constructed the dataset state "these mass-marketed devices provide a flexible, affordable and self-contained solution to automatically and unobtrusively monitor Activities of Daily Living" as a benefit of using smartphones to collect this data.

One down side of using smartphones to collect movement data is they are not consistently located. To the sensors and the model, walking may 'look' different depending on if the phone is held in the hand, front pocket, back pocket, etc. This is where wearable sensors have an advantage as they are usually consistently located on the body - like a wristwatch. However this HAR is deployed, the data will be similar and the goal is to create a robust enough proof of concept model that could move forward into different kinds of deployment.

<img src=images/AdobeStock_367863903.jpeg width=600>

## Data Understanding

This dataset has been taken from the embedded accelerometer and gyroscopes in Samsung Galaxy II smartphones. The tri-axial (x, y, and z) signals were sampled a constant rate of 50Hz (50 samples per second). The sensor signals were then processed through noise filters and then sampled in 2.56 second sliding windows with 50% overlap - this corresponds to 128 readings per window.

Then from each sample window, variables from the time and frequency domain were calculated to obtain a vector of 561 features. It is this 561 feature vector for each window that we use as our data samples to train a model. More details on the specific features and variables can be found in the analysis notebook.

#### Class Breakdown

All samples were then labeled into 6 classes -  sitting, standing, laying, walking, walking upstairs, and walking downstairs. As each indiviual subjects trial varied slightly in length, we dont have exactly even distributions of the classes. But, they do each contain enough samples to model and are close enough to not worry about. Let's examine the class distribution below.

<img src=images/class_distribution.png width=600>

## Modeling

The modeling plan for this project will take advantage of the way our 6 classes happen to be organized. We could choose to group the activites into two groups - dynamic and static - that each contain 3 activities. Performing a 2-stage classification that first performed a dynamic/static binary classification, and then two separate 3-class dynamic and static predictions could greatly increase accuracy and reduce the error in mis-classifiying dynamic and static activities. 

This 2-stage model was proposed in 2018 by researchers at Kookmin University in their 2018 paper and deemed the [Divide and Conquor](https://www.mdpi.com/1424-8220/18/4/1055) method. This analysis seeks to show a proof of concept and the strength of the methods proposed in this paper.

Other considerations for modeling techniques are scalabilty and adaptability. Early HAR methods employed manual feature construction and selection that took much time and effort on the part of the researchers. Adapting these models to even slightly different forms of data became very labor and detail intensive. Given the power of ML tools we have used to now, we wish to employ models that can look at the entire dataset and manually perform feature selection for us. This allows for a more robust model that can be tested on new data without as much labor.

#### First Stage

The first stage model will seek to perform a binary classification on the entire dataset that groups samples into dynamic and static activity groups. As we plan to input the etire 561 feature vector as the input, we want a fast model that can manually select the important stuff. The Kookmin researchers used a decision tree and I will use a random forest. Random forests are decision based models that group the data based on conditions. The way random forests extract the best 'condition' to decide on means that they are an ML method that automatically performs feature selection.

#### Second Stage

Then after data has been grouped into dynamic and static activities, we need two 3-class second stage models that identify the specific activity. Both the dynamic and static second stage models will utilize 1-D Convolutional Nerual Network models. Neural nets are fast and scalable and automatically decide which features in the data are important. Convolutional layers allow us to extract information from not just single features, but patterns found in neighboring features as well. 

## Results / Conclusions

The results for the divide and conquor model were outstanding. The first stage random forest performed with perfect accuracy on the validation data and the static 2nd stage model performed in the high 90s. The dynamic model slightly lacked compared to the others, but still scored well enough for the model to acheive an end-to-end accuracy of 97%. 

We can visualize the results below in a confusion matrix.

<img src=images/cfmatrix.png width=800>

Notice the red boxes in the upper right and bottom left corner. These indicate that there were no mis-classifications in the binary stage. This is huge as we can they rely on more specifically tuned models to the group the specific activties. Also, if we ever want to extract information on time spent doing dynamic vs static activities, we know we can rely on the accuracy of the binary prediction.

## Next Steps

#### Improvements

The Binary and Static models acheived perfect and near perfect accuracy scores. But the Dynamic model was slightly lacking compared to its siblings. I think moving forward, one of the first things to do is tweak the dynamic model until it performs as well as its second stage static counterpart.

Improving the dynamic model could be done by changing the structure and layers of the 1-D CNN, or by adjusting the parameters involved in the data sharpening step. The research paper outlines many possibilites of combinations of successful sharpening parameters, and trying these would be my first step before adjusting the CNN structure.

We also had almost 1000 more training samples in the static model than in the dyanmic model. This also has an effect on how much the model can learn. So improving the dynamic model scores could also come from simply aquiring more data.

#### Streamlining

Since this model so far is a proof of concept for a 2 stage model, it is broken up into 3 different models that must be run separately. Another next step now that we have created good enough models is to pipe them all together end-to-end. While improving and exploring models might be best done on each model separately as I have done here, testing this model on new data or deploying it would be much easier if it were all piped into a single function/model.

#### Deployment Details

Also within the context of deployment, we next need to consider how data will be input and output and what information we want to extract from the model.

Here, single row instances are classified into activity. In practice, we are more likely to input many consecutive samples/rows in time into the model. This would then output a prediction for activity for each window in time. We would then have to decide if we wanted the model to return the most likely or frequent activity for all the data input, or a breakdown of the different activities used. One example for a minute of sensor data could be to classify all rows, choose the most likely activity for each second, and then return information about how much time the user spent doing walking or doing a static activity. This is the kind of information that then becomes actionable to the user and tells them about their activity patterns through the day and how it relates to other health information like heart rate or blood pressure.

<img src=images/AdobeStock_367866890.jpeg width=600>

### Sources / References

Data:
  - [UCI](https://archive.ics.uci.edu/ml/datasets/Smartphone-Based+Recognition+of+Human+Activities+and+Postural+Transitions)
  - [Data Aquisition Paper](https://www.esann.org/sites/default/files/proceedings/legacy/es2013-84.pdf)

Tools / Packages:
  - [Tensorflow](https://www.tensorflow.org/api_docs/python/tf)
  - [Sci-Kit Learn](https://scikit-learn.org/stable/modules/generated/sklearn.ensemble.RandomForestClassifier.html?highlight=random%20forest#sklearn.ensemble.RandomForestClassifier)

References / Sources:
  - [Divide and Conquor HAR Research paper](https://www.mdpi.com/1424-8220/18/4/1055)
  - [Machine Learning Mastery - Summary of the different HAR research methods](https://machinelearningmastery.com/deep-learning-models-for-human-activity-recognition/)
  - [ML Mastery - 1D CNN for HAR](https://machinelearningmastery.com/cnn-models-for-human-activity-recognition-time-series-classification/)


### Thanks for reading!
If you have any questions feel free to contact me via email at jonny.hofmeister@gmail.com
