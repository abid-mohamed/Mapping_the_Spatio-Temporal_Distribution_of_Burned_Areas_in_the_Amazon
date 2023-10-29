# 1. Data Analysis and Missing Data
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