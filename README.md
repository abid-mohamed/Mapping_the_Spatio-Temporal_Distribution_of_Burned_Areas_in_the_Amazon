# Mapping the Spatio-Temporal Distribution of Fires in the Amazon from 2001 to 2020: An Ensemble Modeling Approach

## Overview
This project was conducted during my 6-month internship at King Abdullah University of Science and Technology (KAUST). It was supervised by Prof. Paula Moraga and Dr. Jonatan A. González.

The primary objective of this project was to develop an ensemble modeling approach to predict forest fires in the Amazon rainforest, leveraging data spanning from 2001 to 2020. By integrating various environmental and climatic factors, we aimed to create a comprehensive model that could provide valuable insights into the occurrence of forest fires in this vital ecosystem.

## Data Source
For this project, we utilized the dataset provided by Mateen Mahmood and Prof. Paula Moraga. The dataset is a raster-based resource for spatio-temporal analysis of forest fires in the Amazon rainforest from 2001 to 2020. You can find the data on Zenodo at the following DOI: [Dataset on https://doi.org/10.5281/zenodo.7215402](https://doi.org/10.5281/zenodo.7215402).

## Tools and Packages
To implement this project, we primarily used the R programming language. The following R packages were instrumental in our data analysis, modeling, and visualization: $\texttt{terra}$, $\texttt{raster}$, $\texttt{sf}$, $\texttt{h2o}$, $\texttt{rsample}$, $\texttt{recipes}$, $\texttt{data.table}$, $\texttt{tidyverse}$, $\texttt{pROC}$, $\texttt{doParallel}$, $\texttt{doSNOW}$, $\texttt{ggplot2}$, $\texttt{tidyterra}$. <br />
These packages facilitated various aspects of our project

## Project Steps

This project is structured into distinct steps to facilitate understanding and organization. Each step corresponds to a specific aspect of the analysis and modeling process. You can navigate through the folders corresponding to each step for detailed code and documentation.

### 1. Data Analysis and Missing Data

#### 1.1. Data Analysis 

In this step, we perform an initial data analysis to enhance our understanding of the dataset. This involves exploring the dataset's structure, visualizing important variables, and detecting any missing or unusual data points.

The dataset consists of 10 variables that provide valuable information about fires, land use, environmental conditions, and climate factors. With a spatial resolution of 500 meters, it allows for detailed analysis of the Amazon rainforest. These variables are recorded on a monthly basis, spanning from 2001 to 2020.

#### 1.2. Missing Data

We focus on addressing the significant challenge of identifying and managing missing data within our dataset. We discovered that missing data is predominantly concentrated in two areas:

- The _land surface temperature_ variable, which contains over one million missing data points across twenty-six months.

- The response variable, _burnt area_, in which we treat the value (-2) as missing data, representing water. Additionally, it's important to note that we exclude the two months, September and November 2012, from our analysis, as we do not have any data for the response variable _burnt area_ during these months.

For the remaining covariates, missing data is minimal, consistent across months, and typically located near the map's edges.

[Explore Data Analysis and Missing Data](./1_data_analysis_&_missing_data)

### 2. Data Preparation (Downsampling strategy)

In this step, we address the class imbalance issue identified in the Data Analysis step. To reduce the data imbalance, we employ a down-sampling strategy, which involves the following two key actions:

- For cells with at least one fire event over the 20-year period, we retain all available data records, excluding those with missing values.
 
- For cells without any fire events during this period, we randomly select a single observation from across all the months. It's important to note that this selection ensures that there are no missing values in the _land surface temperature_ covariate or the response variable _burnt area_.

<p align="center">
  <img src="assets/Downsampling_approach.jpg" alt="Downsampling strategy"  width="60%" />
</p>

As the downsampling process still results in a substantial dataset of approximately 550 million observations, we have opted to partition our data into 11 zones, with each zone containing approximately 50 million observations. This division not only reduces the computational load but also enhances the usability and manageability of the dataset.

<p align="center">
  <img src="assets/zones.png"  width="70%" />
</p>

[Explore Data Preparation](./2_data_preparation)

### 3. Model Assessment, Ensemble Model, and Results

In the Model Assessment, Ensemble Model, and Results step, we evaluate the performance of various machine learning models, including Distributed Random Forest (DRF), Generalized Linear Models (GLM), Gradient Boosting Machine (GBM), and eXtreme Gradient Boosting (XGB). Performance assessment is based on two key metrics: AUC (Area Under the Receiver Operating Characteristic Curve) and AUCPR (Area Under the Precision-Recall Curve). These metrics are computed using the $\texttt{h2o}$ package.

Additionally, we create ensemble models for each zone, combining the four methods using a linear combination technique. The weights for each method in the ensemble are determined based on the normalized AUCPR metric. This ensemble approach optimizes predictive accuracy for each zone.

[Explore Model Assessment](./3_model_assessment_&_ensemble_model)

### 4. Maps and Time Trends of Fire Probability

In the "Maps and Time Trends of Fire Probability" section, we present the results of our ensemble model's predictions for fire probabilities in the Amazon rainforest. This section includes two types of visualizations:

**Monthly Fire Probability Maps**

Explore the dynamic monthly fire probability maps through the animated GIF plot below. Each frame in the animation represents a different month across the 20-year study period. This visualization provides valuable insights into the seasonality and the spatial distribution of fire probability in the Amazon.

<p align="center">
  <img src="assets/pred_fire.gif"  width="70%" />
</p>

**Time Trend of Fire Probability**

The boxplots presented below depict the time trend of fire probability during the 20-year study period. These visualizations illustrate how fire risk has evolved over time, offering a comprehensive view of long-term patterns and trends in fire probability.

<p align="center">
  <img src="assets/pred_boxplot.png"  width="70%" />
</p>

These visualizations are powerful tools for understanding the dynamics and patterns of fires in the Amazon rainforest.

---

**Note:** Each folder contains detailed documentation, code, and any additional resources relevant to the corresponding step. You can follow the links to explore each aspect of the project in more detail.
























### About the Data
The dataset includes 10 variables that capture various factors related to fires, land use, environmental conditions, and climate. It provides a spatial resolution of 500 meters, allowing a detailed analysis of the Amazon rainforest. These variables are measured on a monthly basis, covering the entire period from 2001 to 2020.

- **Burnt Area**: Represents the extent of burned areas in the Amazon rainforest, categorized as burnt (1), unburnt (0), missing (-1), or water (-2).

- **Land Cover**: A categorical variable with 11 classes, providing information on different land cover types such as water, urban, forest, grassland, and more.

- **Precipitation**: Measured in millimeters per hour, with a range between 0 and 3300.

- **Soil Moisture**: Measured in millimeters, with missing values marked as -9.99e+08, and a range between 0 and 4291.

- **Elevation**: Measured in meters, with a range between -85 and 6471.

- **Land Surface Temperature**: Represented in Kelvin, with values adjusted by a scale factor of 0.02. Different months have varying missing data.

- **Specific Humidity**: Represented as kg/kg, indicating the ratio of kilograms of water (moisture) per kilogram of air. It ranges from 9.59e-04 to 2.15e-02.

- **Evapotranspiration**: Measured in kg/m2s, with values ranging between -2.02e-07 and 9.69e-05.

- **Wind Speed**: Measured in m/s, with values between 0.86 and 9.85.

- **Air Temperature**: Represented in Kelvin, with values ranging from 268 to 307.

## Handling Missing Data and Data Preparation

### Missing Data
Dealing with missing data is a crucial part of our project. In the context of the "Burnt Area" variable, cells with a value of -2 represent water. However, the status of these cells can change over time to indicate "No fire" (0) or "Fire" (1) in different months. In our analysis, we treat cells with a value of -2 as missing data.

For the covariate "Land Surface Temperature," missing data varies across months. We observed that 26 months had more than 1 million missing data points, with three months exceeding 2 million missing data points. February 2016 had the highest with 3.3 million missing data points. To visually understand these missing data patterns, you can refer to the plots for the months with the most significant gaps.

In contrast, other covariates like "Precipitation," "Soil Moisture," "Specific Humidity," "Evapotranspiration," "Wind Speed," and "Air Temperature" have consistent missing data patterns across all months. These covariates have missing data values mostly at the boundary of the map, and each month's missing data count is below 133,000. We decided to exclude cells with missing data in these covariates as they consistently lacked data throughout the entire study period.

### Data Preparation (Downsampling)
Our dataset features a spatial resolution of 500 meters, resulting in an extensive amount of data. To address the class imbalance in the "Burnt Area" response variable, we employed a careful data preparation strategy known as downsampling. Here's how we balanced the response variable:

1. For cells that experienced at least one fire event during the 238-month study period, we retained all observations for these cells, eliminating any missing data for the corresponding months.

2. For cells that did not experience any fire events during the study period, we randomly selected one observation from each of the 238 months. We ensured that the selected value was not missing data for the covariate "Land Surface Temperature" and was either 0 (unburnt) or 1 (burnt) for the response variable "Burnt Area."

This approach resulted in a dataset containing approximately 550 million observations, covering a substantial portion of the Amazon rainforest. To facilitate further analysis and modeling, we normalized the data and divided it into 11 zones, each with roughly 50 million observations. The zone allocation and data distribution are visualized in Figure 7.

[Insert Figure 7: Visualization of Data Zones and Distribution]

This downsampling strategy not only addresses the class imbalance but also allows for efficient modeling and analysis, providing a balanced and representative dataset for our study.


