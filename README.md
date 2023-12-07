# age-related-BMI-trends
Examining if BMI indicates health decline in low-income groups. Analyzing BMI trends in Argentine adults from 2005 to 2022, using six population surveys. Focus on variations by gender and socioeconomic status


## Introduction
Obesity is recognized as a global epidemic of the 21st century, becoming a significant burden on health systems due to its high association with disease and mortality rates. The Body Mass Index (BMI) is a widely validated tool used to assess the nutritional status of individuals and to project public health policies. Recently, high BMI levels have been on the rise in Argentina, particularly among groups with less economic resources. This study explores the interrelatedness among BMI, socioeconomic status, and health across various age groups. The significance of comprehending these correlations in the context of Argentina's ongoing economic difficulties is emphasized. The investigation is established on an extensive sample that spans from 2005 to the current day, analyzing the province of Córdoba's adult population. The purpose is to examine trends in BMI and their correlation with age and socioeconomic status, while contrasting them with the general Argentine population during diverse time periods.

## Methods

**Data source** The study utilized data from two sources: the Córdoba Obesity and Diet Study (CODIES) and National Risk Factors Surveys (NRFS). CODIES I and II, conducted in Córdoba in 2006 and 2022, respectively, employed a cross-sectional design with random sampling. CODIES II enrolled 1327 subjects, and both studies covered individuals aged 18-85. NRFS, a government tool for data collection, evaluated risk factors in the Argentine population in 2005, 2009, 2013, and 2018 (n=16,577 for anthropometrics). Trained nutritionists gathered information on demographics, economic status, health, and dietary habits, while utilizing the International Physical Activity Questionnaire to gather data on physical activity. Ethical approval was obtained from the appropriate committees.

**Descriptive statistics** analyzed continuous data and BMI trends based on time, gender, age, and SES using Python and Stata. Median moving averages (MMA) were calculated with a lag of age ±1. The first derivative of the quadratic function of age determined the age at which maximum BMI occurred. Statistical analyses were performed with Python v3.11.4, and graphics were created using Stata v17. The analysis involved generating scatterplots of BMI by age and calculating simple moving averages with a three-year width. Median BMI values were used to fit a second-degree polynomial curve, and measures of kurtosis, standard deviation, and skewness were examined. The analyses were carried out on the overall population and then separated by gender and SES (low, medium, high).

## 3 Code

### **BMI Trajectories**

This function, bmi_trajectories, analyzes BMI trajectories utilizing quadratic curve fitting across different age intervals. It calculates statistical measures including kurtosis, standard deviation, skewness, mean, and median of BMI at each specific age interval. Furthermore, it applies a quadratic curve to the median BMI values, providing coefficients and corresponding errors. Additionally, this function computes the age of maximal BMI, maximum BMI value, and their respective errors. The mean fluctuation measures the difference between the actual and predicted median BMI values. The function then visualizes these analyses through subplots and displays essential coefficients and statistical measures for further interpretation.

    import pandas as pd
    import numpy as np
    import matplotlib.pyplot as plt
    from scipy.stats import entropy

    # Function to calculate BMI trajectories by age and plot various statistics
    def bmi_trajectories(df, age_column_name, bmi_column_name, title):
      # Lists to store data points
      ages = []
      kurtosis_values = []
      std_values = []
      skew_values = []
      mean_values = []
      median_values = []
      entropy_values = []
      fluctuation = []

      # Loop through age range
      for n in range(18, 85):
          # Filter data for the current age and two consecutive ages
          filtered_data = df[(df[age_column_name] == n) | (df[age_column_name] == n + 1) | (df[age_column_name] == n + 2)]
          ages.append(n)
          kurtosis_values.append(filtered_data[bmi_column_name].kurtosis())
          std_values.append(filtered_data[bmi_column_name].std())
          skew_values.append(filtered_data[bmi_column_name].skew())
          mean_values.append(filtered_data[bmi_column_name].mean())
          median_values.append(filtered_data[bmi_column_name].median())
          entropy_values.append(entropy(filtered_data[bmi_column_name].astype(float)))
  
      # Create subplots
      fig, axs = plt.subplots(nrows=2, ncols=2, figsize=(8, 6))
  
      # Plot STD
      axs[0, 1].plot(ages, std_values, 'o', color='C3', label='STD')
      axs[0, 1].set_title('STD')
  
      # Plot Kurtosis
      axs[1, 1].plot(ages, kurtosis_values, 'o', color='C4', label='Kurtosis')
      axs[1, 1].set_title('Kurtosis')
  
      # Plot Skew
      axs[1, 0].plot(ages, skew_values, 'o', color='C5', label='Skew')
      axs[1, 0].set_title('Skew')
  
      plt.subplots_adjust(wspace=0.3, hspace=0.3)
      plt.legend()
  
      # Plot Mean and Median
      axs[0, 0].plot(ages, mean_values, 'o', color='C2', label='Mean')  # green
      axs[0, 0].plot(ages, median_values, 'o', color='C0', label='Median')  # blue
      axs[0, 0].set_title('Mean and Median')
  
      # Fit a quadratic curve to the median BMI values
      xfit = ages[0:70]
      yfit = median_values[0:70]
  
      coeffs, cov = np.polyfit(xfit, yfit, 2, cov=True)
      fitted_curve = np.polyval(coeffs, ages)
      error_fit = np.sqrt(np.diag(cov))
  
      for n in range(0, len(xfit)):
          fluctuation.append(abs(yfit[n] - coeffs[0] * xfit[n] ** 2 - coeffs[1] * xfit[n] - coeffs[2]))
  
      # Plot the fitted curve
      axs[0, 0].plot(ages, fitted_curve, label='Fitted Curve', color='C1')
  
      # Print coefficients and errors
      print(f'Age of Maximum BMI: {(-coeffs[1] / (2 * coeffs[0])).round(2)} years')
      print(f'Error in Age of Maximum BMI: {np.sqrt((error_fit[1] / (2 * coeffs[0]))**2 + (error_fit[0] * coeffs[1] / (2 * coeffs[0]**2))**2)}')
      print(f'Quadratic Coefficient (a): {-coeffs[0]}')
      print(f'Maximum BMI: {(-coeffs[1]**2 / (4 * coeffs[0]) + coeffs[2])}')
      print(f'Error in Maximum BMI: {np.sqrt(error_fit[2]**2 + (coeffs[1] * error_fit[1] / (2 * coeffs[0]))**2 + (error_fit[0] * coeffs[1]**2 / (4 * coeffs[0]**2))**2)}')
      print(f'Mean Fluctuation: {np.median(fluctuation)}')
  
      print(f'Quadratic Coefficients (a-b-c): [{coeffs[0]:.8f}, {coeffs[1]:.8f}, {coeffs[2]:.8f}]')
      print(f'Error in Coefficients (err a, b, c): [{error_fit[0]:.8f}, {error_fit[1]:.8f}, {error_fit[2]:.8f}]')
      print(f'Coefficients: {coeffs}')
      print(f'Error in Coefficients: {error_fit}')
      fig.suptitle(title)
      plt.show()
  
      # Plot Entropy
      plt.plot(ages, entropy_values, 'o', color='C2', label='Entropy')  # green
      plt.title('Entropy')
      plt.show()

We utilized the BMI trajectories function to create plots and gather statistics for each population sample without distinguishing by any other variables.  The trajectories were determined for the entire population in every dataset and are defined as follows:
      
    # Base1
    bmi_trajectories(df, 'age', 'bmi',"BMI-age distribution - base1")

We filtered the function to plot BMI curves by gender for each population.

    # base1 by Gender
    bmi_trajectories(df[(df['gender']==1)],'age','bmi') #Male
    bmi_trajectories(df[(df['gender']==2)],'age','bmi') #Female

Then the same procedure was carried out, but this time taking into consideration the socio-economic stratum (SES).

    # base1 by SES
    bmi_trajectories(df[(df['ses']==1)],'age','bmi') #low SES
    bmi_trajectories(df[(df['ses']==2)],'age','bmi') #middle SES
    bmi_trajectories(df[(df['ses']==3)],'age','bmi') #high SES

Finally, the analysis was expanded to include graphing the BMI curves for each socioeconomic status and sex, resulting in 6 graphs for each population under study. The graphs are presented below.

    # base1 by Male gender 
    bmi_trajectories(df[(df['gender'] == 1) & (df['ses'] == 1)], 'age', 'bmi') #low SES men
    bmi_trajectories(df[(df['gender'] == 1) & (df['ses'] == 2)], 'age', 'bmi') #middle SES men
    bmi_trajectories(df[(df['gender'] == 1) & (df['ses'] == 3)], 'age', 'bmi') #high SES men

    # base1 by Female gender
    bmi_trajectories(df[(df['gender'] == 2) & (df['ses'] == 1)], 'age', 'bmi') #low SES women
    bmi_trajectories(df[(df['gender'] == 2) & (df['ses'] == 2)], 'age', 'bmi') #middle SES women
    bmi_trajectories(df[(df['gender'] == 2) & (df['ses'] == 3)], 'age', 'bmi') #high SES women

## Results

In CODIES, about 42.5% of participants were women, and in NRFS, the proportion ranged around 45%, except for 2009 when it hit a high of 65.9% for women. The majority of participants in both studies were between the ages of 30 and 60. Education levels experienced a decrease in the lowest category by nearly 8% from 2005 to 2018 nationwide, with comparable trends in CODIES I and II. Socioeconomic status remained constant, with 80% consistently in the middle and low brackets throughout all studied years. The average BMI among populations surpassed 25 kg/m2, with a slight upward trend observed in both NRFS and CODIES versions, reaching 27.5 kg/m2 in 2018. The prevalence of obesity in NRFS increased from 16% in 2005 to nearly 29% in 2018, implying a doubled trend. A similar pattern occurred in CODIES I and II. 

### **Main bio-socio characteristics of studied populations: NRFS (2005, 2009, 2013, 2018); CODIES (I and II)**

![image](https://github.com/steffstaurini/age-related-BMI-trends/assets/99434841/b4e07d46-6976-4363-bcb7-bbc343a0a061)

// los resultados gruesos de las curvas todavia no los pongo por las dudas
// conclusiones lo mismo

## conclusion
// principales conclusiones

## consideraciones finales
// autores, link a full text, etc
