# Technical Report - Drinking Supply Water Contamination Classification

The California Dept of Public Health estimates that 85 percent of California’s community public water systems, supplying more than 30 million residents, rely on groundwater for at least part of their drinking water supply. Because many community water systems are entirely reliant on groundwater for their drinking water supply, contamination of this resource can have  far-reaching consequences. Many groundwater basins throughout California are contaminated with either naturally occurring or anthropogenic pollutants, or both. When a groundwater source is contaminated, community water systems must use costly treatment systems to ensure that the water is safe to drink. Where treatment and alternative water supplies are not available, some community water systems serve contaminated groundwater until a solution is implemented. 
Contamination of the state’s groundwater resources results in higher costs for ratepayers and consumers due to the necessity of additional treatment and can pose a threat to public health for community water systems that cannot afford the necessary treatment systems. The identification of a relationship between ambient groundwater quality and community water systems that rely on a given groundwater source may help focus available efforts and resources to ensure the provision of safe drinking water.

## Problem Statement

Due to the enormity of the data available, the scope of this project was selected to pertain to samples collected from Kern County from 2012 - present. This is a follow up to a [legislative report](https://www.waterboards.ca.gov/water_issues/programs/gama/ab2222/docs/ab2222.pdf) published in 2013 about communities in California that rely on contaminated groundwater as a water source, identifying Kern County as having the most community water systems that are 100% reliant on groundwater. 

![gw reliant counties](https://github.com/nmlook/Groundwater_Quality_GACapstone/blob/master/assets/Visuals/AB2222_Counties_Reliant_on_GW.PNG)

It is the aim of this project to build a classification model to quantify the association between groundwater monitoring results from the Groundwater Ambient Monitoring and Assessment (GAMA) and the California Dept of Public Health’s drinking water quality results (DDW). For a classification model, the confusion matrix metric will be used to evaluate the model’s performance. Since detection of contamination in a system’s supply is a matter of public health, optimization should be geared towards sensitivity, the rate of true positives. 

## Data Collection

Data collected for the purpose of this project was obtained from three departments on the California State Water Board site: The DDW’s [Electronic Data Transfer Library](https://www.waterboards.ca.gov/drinking_water/certlic/drinkingwater/EDTlibrary.html) and the GAMA’s [Data Download site](http://geotracker.waterboards.ca.gov/gama/datadownload), and [GeoTracker](http://geotracker.waterboards.ca.gov/map/). For drinking water, a total of four datasets were necessary to download, convert into excel files and loaded into a Jupyter Notebook with the pandas library. 
The chemical.zip file contains chemical data from January 2012 to the current month for all California’s water quality monitoring. The Siteloc.zip file contains information regarding each well station, the relevant information for these purposes includes which system it is associated with, the water type, and its status. The Storet.zip file lists each chemical code number and its units, MCL (Maximum Contaminant Level), and other levels pertaining to monitoring. The Watsys.zip file contains each Water Systems identification number, name, and address though it was largely unused for the purposes of this project. In the future, it would be useful to use it for describing the findings to particular communities or areas. Data exported from the GeoTracker website was used to map each drinking water well station with a location. 
For the groundwater, only Kern County’s data was needed, both available as text files. One file was named KernEDF, which contained all ambient groundwater monitoring data. The other file named KernGeoXY contained well location information and description.

## Exploratory Data Analysis
Several data munging decisions were made during this phase to align with the findings in the legislative report, such as to only include active raw and active untreated wells and then to further narrow chemical results to what the legislative report names as Principal Contaminants. 

Once each individual dataset were cleaned of structural errors and irrelevant information, all containing descriptive information about the well location were transformed into dictionaries and mapped to add on to the datasets containing water quality results, one for drinking water and one for groundwater. Using pairwise distancing across latitude, longitude, and date. 

The DDW strongly cautions users to use care when interpreting data made available, as the presence of a contaminant in raw water at a given concentration does not necessarily mean that the water was served by the water system to its customers, or, if served, that the contaminant was present at that concentration. However, in the interest of time and simplicity, any result over a chemical’s established maximum contaminant level (MCL) was considered to be a positive class.


**Kern County and its 263 water systems**
![Kern sys](https://github.com/nmlook/Groundwater_Quality_GACapstone/blob/master/assets/Visuals/Kern_system.PNG)

**Groundwater contaminants over time**
![gw contaminants](https://github.com/nmlook/Groundwater_Quality_GACapstone/blob/master/assets/Visuals/groundwater_princip_contaminants.PNG)

<<Add Drinking Water Over MCL Detections Here>>


The imbalance of classes proved to be a significant problem later on in modeling, as the baseline for Nitrate, Fluoride, Alpha radiation, Uranium, and Perchlorate were all below 10%. Tetrachloroethylene did not have any positive classes and therefore could not be predicted or inferred on. Arsenic had the least imbalanced class at a 21% positive class percentage.

## Modeling

Interpretability and readability is an important factor in selecting a model because transparency is a highly regarded value as a state agency. Black-box models might not be entirely appropriate as a final selection, so a Decision Tree, a Random Forest, and a Logistic Regression Classification model were chosen. There was a realization that the nature of the problem is not simply one of mutually exclusive classification. A sample of drinking water can have more than one contaminant presenting as a hit above its maximum limit. Therefore, each contaminant needed be modeled separately.
Class imbalance was a significant challenge that, though oversampling was performed, could not be understated as a major hurdle to overcome. No optimization was performed on any of the three types of models and should also be listed as a followup endeavor, given more time and more thought to how to overcome the class imbalance.

## Results

The Decision Tree Classifier with all default settings performed with a training accuracy between 78.3% - 98.6% and a testing accuracy between 79.6% - 98.7% across all contaminants detected over MCL. While the accuracy may look impressive on Nitrate, Alpha radiation, Perchlorate, and both measures of Uranium, the confusion matrix tells a different story. The Decision Tree model was only able to classify 12 true positives for Arsenic and next to none for each of the other contaminants. What is most concerning are false negatives here, 56 in the case of Arsenic, as these represent contaminated water sent to a treatment plant having been initially believed to be clean.

![decision tree](https://github.com/nmlook/Groundwater_Quality_GACapstone/blob/master/assets/Visuals/default_DTree_on_AS_oversamp_top.PNG)

The Random Forest Classifier with all default settings performed with a training accuracy between 78.1% - 98.8% and a testing accuracy between 79.7% - 99.0%. Which is nearly identical to the performance of a single Decision Tree. There is less evidence of overfitting due to the training accuracy being slightly higher than the testing accuracy. However, the confusion matrix also shows that these Random Forest Classifier models also suffer from low true positive rates and not-negligible false negative rates.
The Logistic Regression Classifier with all default settings performed with a training accuracy in the high 70 percentage for Arsenic while all the other chemicals were in the high 90 percentage. This follows the same ranges as the Decision Tree and Random Forest models previous. There is less evidence here of overfitting due to the training accuracy being nearly identical to the testing accuracy. However, it performed the worst of the models in terms of precision for Arsenic, the least imbalanced class.

**Logistic Regression Beta Coefficients**
![beta coefs](https://github.com/nmlook/Groundwater_Quality_GACapstone/blob/master/assets/Visuals/log_reg_beta_coefficients.PNG)

The table above is of the Beta Coefficients for each groundwater chemical measurement to drinking water detection over MCL. What is most notable is that for each one microgram per Liter increase of gross alpha counts of radionuclides in groundwater there is a 0.0078 pCi/L fo Uranium in the nearest drinking water well station. There is also a 0.0077 increase in units of another method of measuring Uranium in the drinking water for a unit increase Uranium detected in the groundwater. The Beta coefficients are so miniscule as to nearly be negligent or too small to be considered within significant figures.

## Future Steps
The first next step would be to use Standard Scaling on Latitude, Longitude, and date of each sampling to more accurately weight the nearest neighbor of a groundwater well to one of a drinking water supply well by first physical location, then sampling date. Another step to take in that same stage would be to use more feature engineering to have several nearby groundwater wells inform on a drinking water supply well as opposed to only one. After that, model optimization could be done to get better performance from each model type.

More subject area expertise is required for better decision making in the process of extracting both more and more informative data from the raw files available to the public. As well as an engaged interaction with the State Water Quality Control Board as it continues to encourage more citizen engagement and innovation with its increasingly more transparent data opened to the public. Feedback to the Board could create change towards a cleaner, less convoluted set of datasets to operate on in the future, lowering the barrier towards analysis and exploration.

There is also promising research in the use of neural networks and support vector kernels for use in the environmental sciences such as this field. So the model selection could be expanded further to evaluate the performance of those on modeling. There is a significant geospatial and temporal correlation element to the data that I was unable to explore during this time. So another set of models to consider would be one of geospatial and ARIMA.
