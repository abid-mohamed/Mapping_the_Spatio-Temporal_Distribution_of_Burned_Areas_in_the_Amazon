[*<< II. Data Preparation*](https://github.com/abid-mohamed/Mapping_the_Spatio-Temporal_Distribution_of_Fires_in_the_Amazon/blob/main/2_data_preparation/README.md) 
&emsp; | &emsp;
[*IV. Maps and Time Trends of Fire Probability >>*](https://github.com/abid-mohamed/Mapping_the_Spatio-Temporal_Distribution_of_Fires_in_the_Amazon/blob/main/README.md#iv-maps-and-time-trends-of-fire-probability)

# III. Models, Ensemble Model and Models Assessment

## III.1. Models

In this section, we utilize the $\texttt{h2o}$ package on the training data across all zones to deploy a diverse set of machine learning models, encompassing Distributed Random Forest (DRF), Generalized Linear Models (GLM), Gradient Boosting Machine (GBM), and eXtreme Gradient Boosting (XGB).

To optimize the performance of each method, we engage in meticulous hyperparameter tuning through a traditional (or "cartesian") grid search. This involves specifying crucial parameters for each model, enabling $\texttt{h2o}$ to systematically evaluate the 'AUC' metric by training a model for every combination of hyperparameter values.

##  III.2. Hyperparameters Tuning

For the hyperparameter tuning process, we adopt a 10-fold cross-validation approach. Given the imbalanced distribution of the response variable *Burnt Area*, maintaining a balanced proportion of cells with 'No Fire' event (0) and cells with 'Fire' event (1) within each fold is crucial. To achieve this balance, we utilize the `vfold_cv()` function from the $\texttt{rsample}$ package, employing the `strata = BurntArea` option for a stratified split.

Acknowledging the time-consuming nature of this technique, we strategically apply it in selected zones. Following fine-tuning, the carefully chosen hyperparameters for each model are uniformly applied to construct models across all zones, ensuring robust and consistent performance throughout the entire dataset.

## III.3. Models Assessment

To assess the performance of each model across all zones, we apply each trained model to the test data of its respective zone, combined with the remaining zones.

The 'AUC' metric is a standard choice for evaluating model performance in scenarios with balanced data. However, given the imbalanced nature of our data, and particularly when false positives carry higher costs than false negatives, we favor the 'AUCPR' metric. This metric proves to be the most suitable choice for evaluating model performance, thanks to its ability to effectively capture relevant positive instances.

## III.4. Ensemble Model

For each zone, we construct an ensemble model by computing a linear combination of the four individual models, assigning weights based on normalized 'AUCPR' values. The final ensemble model is derived by averaging the outputs of all individual ensemble models.

## III.5. Result of Methods Assessment

The table below presents the calculated performance metrics for various models and ensemble models across all zones:

<!-- Table -->
| **Zone** | **GLM AUC** | **GLM AUCPR** | **GBM AUC** | **GBM AUCPR** | **XGB AUC** | **XGB AUCPR** | **DRF AUC** | **DRF AUCPR** | **Ensemble AUC** | **Ensemble AUCPR** |
|:--------:|:-----------:|:-------------:|:-----------:|:---------------:|-------------|---------------|-------------|---------------|------------------|---------------------|
| 1        | 0.862       | 0.061         | 0.864       | 0.061         | 0.769       | 0.024         | 0.861       | 0.074         | 0.875            | 0.088               |
| 2        | 0.710       | 0.027         | 0.846       | 0.041         | 0.723       | 0.027         | 0.849       | 0.061         | 0.839            | 0.038               |
| 3        | 0.864       | 0.057         | 0.859       | 0.055         | 0.804       | 0.038         | 0.863       | 0.078         | 0.945            | 0.158               |
| 4        | 0.793       | 0.061         | 0.850       | 0.051         | 0.787       | 0.029         | 0.834       | 0.077         | 0.922            | 0.156               |
| 5        | 0.839       | 0.057         | 0.855       | 0.048         | 0.773       | 0.027         | 0.839       | 0.075         | 0.907            | 0.163               |
| 6        | 0.825       | 0.057         | 0.856       | 0.051         | 0.776       | 0.030         | 0.833       | 0.080         | 0.925            | 0.168               |
| 7        | 0.767       | 0.040         | 0.817       | 0.047         | 0.744       | 0.021         | 0.818       | 0.063         | 0.806            | 0.035               |
| 8        | 0.798       | 0.030         | 0.824       | 0.035         | 0.749       | 0.024         | 0.833       | 0.075         | 0.885            | 0.157               |
| 9        | 0.798       | 0.037         | 0.827       | 0.036         | 0.765       | 0.026         | 0.829       | 0.047         | 0.904            | 0.081               |
| 10       | 0.721       | 0.019         | 0.857       | 0.051         | 0.788       | 0.032         | 0.849       | 0.067         | 0.927            | 0.108               |
| 11       | 0.797       | 0.047         | 0.841       | 0.056         | 0.713       | 0.026         | 0.826       | 0.081         | 0.959            | 0.299               |




<!-- Table -->
<table align="center">
  <tr>
    <th rowspan="2">Zone</th>
    <th colspan="2">GLM</th>
    <th colspan="2">GBM</th>
    <th colspan="2">XGB</th>
    <th colspan="2">DRF</th>
    <th colspan="2">Ensemble</th>
  </tr>
  <tr>
    <th>AUC</th>
    <th>AUCPR</th>
    <th>AUC</th>
    <th>AUCPR</th>
    <th>AUC</th>
    <th>AUCPR</th>
    <th>AUC</th>
    <th>AUCPR</th>
    <th>AUC</th>
    <th>AUCPR</th>
  </tr>
  <tr align="center">
    <td style="font-size:12px">1</td>
    <td style="font-size:12px">0.862</td>
    <td style="font-size:12px">0.061</td>
    <td style="font-size:12px">0.864</td>
    <td style="font-size:12px">0.061</td>
    <td style="font-size:12px">0.769</td>
    <td style="font-size:12px">0.024</td>
    <td style="font-size:12px">0.861</td>
    <td style="font-size:12px">0.074</td>
    <td style="font-size:12px">0.875</td>
    <td style="font-size:12px">0.088</td>
  </tr>
  <tr align="center">
    <td style="font-size:12px">2</td>
    <td style="font-size:12px">0.710</td>
    <td style="font-size:12px">0.027</td>
    <td style="font-size:12px">0.846</td>
    <td style="font-size:12px">0.041</td>
    <td style="font-size:12px">0.723</td>
    <td style="font-size:12px">0.027</td>
    <td style="font-size:12px">0.849</td>
    <td style="font-size:12px">0.061</td>
    <td style="font-size:12px">0.839</td>
    <td style="font-size:12px">0.038</td>
  </tr>
  <tr align="center">
    <td style="font-size:12px">3</td>
    <td style="font-size:12px">0.864</td>
    <td style="font-size:12px">0.057</td>
    <td style="font-size:12px">0.859</td>
    <td style="font-size:12px">0.055</td>
    <td style="font-size:12px">0.804</td>
    <td style="font-size:12px">0.038</td>
    <td style="font-size:12px">0.863</td>
    <td style="font-size:12px">0.078</td>
    <td style="font-size:12px">0.945</td>
    <td style="font-size:12px">0.158</td>
  </tr>
  <tr align="center">
    <td style="font-size:12px">4</td>
    <td style="font-size:12px">0.793</td>
    <td style="font-size:12px">0.061</td>
    <td style="font-size:12px">0.850</td>
    <td style="font-size:12px">0.051</td>
    <td style="font-size:12px">0.787</td>
    <td style="font-size:12px">0.029</td>
    <td style="font-size:12px">0.834</td>
    <td style="font-size:12px">0.077</td>
    <td style="font-size:12px">0.922</td>
    <td style="font-size:12px">0.156</td>
  </tr>
  <tr align="center">
    <td style="font-size:12px">5</td>
    <td style="font-size:12px">0.839</td>
    <td style="font-size:12px">0.057</td>
    <td style="font-size:12px">0.855</td>
    <td style="font-size:12px">0.048</td>
    <td style="font-size:12px">0.773</td>
    <td style="font-size:12px">0.027</td>
    <td style="font-size:12px">0.839</td>
    <td style="font-size:12px">0.075</td>
    <td style="font-size:12px">0.907</td>
    <td style="font-size:12px">0.163</td>
  </tr>
  <tr align="center">
    <td style="font-size:12px">6</td>
    <td style="font-size:12px">0.825</td>
    <td style="font-size:12px">0.057</td>
    <td style="font-size:12px">0.856</td>
    <td style="font-size:12px">0.051</td>
    <td style="font-size:12px">0.776</td>
    <td style="font-size:12px">0.030</td>
    <td style="font-size:12px">0.833</td>
    <td style="font-size:12px">0.080</td>
    <td style="font-size:12px">0.925</td>
    <td style="font-size:12px">0.168</td>
  </tr>
  <tr align="center">
    <td style="font-size:12px">7</td>
    <td style="font-size:12px">0.767</td>
    <td style="font-size:12px">0.040</td>
    <td style="font-size:12px">0.817</td>
    <td style="font-size:12px">0.047</td>
    <td style="font-size:12px">0.744</td>
    <td style="font-size:12px">0.021</td>
    <td style="font-size:12px">0.818</td>
    <td style="font-size:12px">0.063</td>
    <td style="font-size:12px">0.806</td>
    <td style="font-size:12px">0.035</td>
  </tr>
  <tr align="center">
    <td style="font-size:12px">8</td>
    <td style="font-size:12px">0.798</td>
    <td style="font-size:12px">0.030</td>
    <td style="font-size:12px">0.824</td>
    <td style="font-size:12px">0.035</td>
    <td style="font-size:12px">0.749</td>
    <td style="font-size:12px">0.024</td>
    <td style="font-size:12px">0.833</td>
    <td style="font-size:12px">0.075</td>
    <td style="font-size:12px">0.885</td>
    <td style="font-size:12px">0.157</td>
  </tr>
  <tr align="center">
    <td style="font-size:12px">9</td>
    <td style="font-size:12px">0.798</td>
    <td style="font-size:12px">0.037</td>
    <td style="font-size:12px">0.827</td>
    <td style="font-size:12px">0.036</td>
    <td style="font-size:12px">0.765</td>
    <td style="font-size:12px">0.026</td>
    <td style="font-size:12px">0.829</td>
    <td style="font-size:12px">0.047</td>
    <td style="font-size:12px">0.904</td>
    <td style="font-size:12px">0.081</td>
  </tr>
  <tr align="center">
    <td style="font-size:12px">10</td>
    <td style="font-size:12px">0.721</td>
    <td style="font-size:12px">0.019</td>
    <td style="font-size:12px">0.857</td>
    <td style="font-size:12px">0.051</td>
    <td style="font-size:12px">0.788</td>
    <td style="font-size:12px">0.032</td>
    <td style="font-size:12px">0.849</td>
    <td style="font-size:12px">0.067</td>
    <td style="font-size:12px">0.927</td>
    <td style="font-size:12px">0.108</td>
  </tr>
  <tr align="center">
    <td style="font-size:12px">11</td>
    <td style="font-size:12px">0.797</td>
    <td style="font-size:12px">0.047</td>
    <td style="font-size:12px">0.841</td>
    <td style="font-size:12px">0.056</td>
    <td style="font-size:12px">0.713</td>
    <td style="font-size:12px">0.026</td>
    <td style="font-size:12px">0.826</td>
    <td style="font-size:12px">0.081</td>
    <td style="font-size:12px">0.959</td>
    <td style="font-size:12px">0.299</td>
  </tr>
</table>





[*<< II. Data Preparation*](https://github.com/abid-mohamed/Mapping_the_Spatio-Temporal_Distribution_of_Fires_in_the_Amazon/blob/main/2_data_preparation/README.md) 
&emsp; | &emsp;
[*IV. Maps and Time Trends of Fire Probability >>*](https://github.com/abid-mohamed/Mapping_the_Spatio-Temporal_Distribution_of_Fires_in_the_Amazon/blob/main/README.md#iv-maps-and-time-trends-of-fire-probability)