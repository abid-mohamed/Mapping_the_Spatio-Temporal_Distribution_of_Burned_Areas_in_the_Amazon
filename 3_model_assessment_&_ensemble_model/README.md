[*<< II. Data Preparation*](https://github.com/abid-mohamed/Mapping_the_Spatio-Temporal_Distribution_of_Fires_in_the_Amazon/blob/main/2_data_preparation/README.md) 
&emsp; | &emsp;
[*IV. Maps and Time Trends of Fire Probability >>*](https://github.com/abid-mohamed/Mapping_the_Spatio-Temporal_Distribution_of_Fires_in_the_Amazon/blob/main/README.md#iv-maps-and-time-trends-of-fire-probability)

# III. Models, Ensemble Model and Methods Assessment

In this section, utilizing the $\texttt{h2o}$ package on the training data across all zones, we employ a diverse set of machine learning models, including Distributed Random Forest (DRF), Generalized Linear Models (GLM), Gradient Boosting Machine (GBM), and eXtreme Gradient Boosting (XGB).

To fine-tune the performance of each method, we conduct a meticulous hyperparameter tuning process through a traditional (or "cartesian") grid search. This involves specifying a set of the most crucial parameters for each model, with $\texttt{h2o}$ systematically evaluating the 'AUC' metric by training a model for every combination of hyperparameter values.

This optimization process is executed using a 10-folds cross-validation approach. Acknowledging the time-consuming nature of this technique, we strategically apply it in selected zones. Subsequently, the carefully chosen hyperparameters for each model are uniformly applied to construct models across all zones, ensuring robust and consistent performance across the entire dataset.


---


In this section, utilizing the $\texttt{h2o}$ package on the training data across all zones, we employ a diverse set of machine learning models, including Distributed Random Forest (DRF), Generalized Linear Models (GLM), Gradient Boosting Machine (GBM), and eXtreme Gradient Boosting (XGB).

To optimize the performance of each method, we engage in a meticulous hyperparameter tuning process through a traditional (or "cartesian") grid search. This involves specifying a set of crucial parameters for each model, allowing $\texttt{h2o}$ to systematically evaluate the 'AUC' metric by training a model for every combination of hyperparameter values.

Executing this optimization process involves a 10-folds cross-validation approach. Given the imbalanced distribution of the response variable *Burnt Area*, maintaining a balanced proportion of cells with 'No Fire' event (0) and cells with 'Fire' event (1) within each fold is crucial. To achieve this balance, the `vfold_cv()` function from the $\texttt{rsample}$ package is employed, leveraging the `strata = BurntArea` option for a stratified split.

Acknowledging the time-consuming nature of this technique, we strategically apply it in selected zones. Following fine-tuning, the carefully chosen hyperparameters for each model are uniformly applied to construct models across all zones, ensuring robust and consistent performance throughout the entire dataset.


## III.1. Hyperparameters Tuning

## III.2. Models

### Gradient Boosting Machine (GBM)

### Distributed Random Forest (DRF)

### Generalized Linear Models (GLM)

### Extreme Gradient Boosting (XGBoost)

## III.3. Ensemble Model and Performance

### Compute the weigh of each model

### Ensemble Model

## III.4. Methods Assessment



[*<< II. Data Preparation*](https://github.com/abid-mohamed/Mapping_the_Spatio-Temporal_Distribution_of_Fires_in_the_Amazon/blob/main/2_data_preparation/README.md) 
&emsp; | &emsp;
[*IV. Maps and Time Trends of Fire Probability >>*](https://github.com/abid-mohamed/Mapping_the_Spatio-Temporal_Distribution_of_Fires_in_the_Amazon/blob/main/README.md#iv-maps-and-time-trends-of-fire-probability)