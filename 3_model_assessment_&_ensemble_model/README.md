[*<< II. Data Preparation*](https://github.com/abid-mohamed/Mapping_the_Spatio-Temporal_Distribution_of_Fires_in_the_Amazon/blob/main/2_data_preparation/README.md) 
&emsp; | &emsp;
[*IV. Maps and Time Trends of Fire Probability >>*](https://github.com/abid-mohamed/Mapping_the_Spatio-Temporal_Distribution_of_Fires_in_the_Amazon/blob/main/README.md#iv-maps-and-time-trends-of-fire-probability)

# III. Models, Ensemble Model and Methods Assessment

## III.1. Models

In this section, we utilize the $\texttt{h2o}$ package on the training data across all zones to deploy a diverse set of machine learning models, encompassing Distributed Random Forest (DRF), Generalized Linear Models (GLM), Gradient Boosting Machine (GBM), and eXtreme Gradient Boosting (XGB).

To optimize the performance of each method, we engage in meticulous hyperparameter tuning through a traditional (or "cartesian") grid search. This involves specifying crucial parameters for each model, enabling $\texttt{h2o}$ to systematically evaluate the 'AUC' metric by training a model for every combination of hyperparameter values.

##  III.2. Hyperparameters Tuning

For the hyperparameter tuning process, we adopt a 10-fold cross-validation approach. Given the imbalanced distribution of the response variable *Burnt Area*, maintaining a balanced proportion of cells with 'No Fire' event (0) and cells with 'Fire' event (1) within each fold is crucial. To achieve this balance, we utilize the `vfold_cv()` function from the $\texttt{rsample}$ package, employing the `strata = BurntArea` option for a stratified split.

Acknowledging the time-consuming nature of this technique, we strategically apply it in selected zones. Following fine-tuning, the carefully chosen hyperparameters for each model are uniformly applied to construct models across all zones, ensuring robust and consistent performance throughout the entire dataset.

## III.2. Ensemble Model and Performance

### Compute the weigh of each model

### Ensemble Model

## III.3. Methods Assessment



[*<< II. Data Preparation*](https://github.com/abid-mohamed/Mapping_the_Spatio-Temporal_Distribution_of_Fires_in_the_Amazon/blob/main/2_data_preparation/README.md) 
&emsp; | &emsp;
[*IV. Maps and Time Trends of Fire Probability >>*](https://github.com/abid-mohamed/Mapping_the_Spatio-Temporal_Distribution_of_Fires_in_the_Amazon/blob/main/README.md#iv-maps-and-time-trends-of-fire-probability)