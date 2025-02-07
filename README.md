# age-related-BMI-trends
Examining if BMI indicates health decline in low-income groups. Analyzing BMI trends in Argentine adults from 2005 to 2022, using six population surveys. Focus on variations by gender and socioeconomic status

## Methodology

* This research combines data from two sources:
Córdoba Obesity and Diet Study (CODIES): Cross-sectional surveys conducted in Córdoba, Argentina (CODIES I: 2006, CODIES II: 2022). Both enrolled subjects aged 18-85 years using random samples from the general population.
National Risk Factors Surveys (NRFS): A national survey conducted by the Argentinian government across multiple years (2005, 2009, 2013, 2018). It employed a stratified, multistage approach to collect data from the general population aged 18 or older.

**Statistical Analyses**
* This study focused on specific data points: age (continuous and categorical), sex, educational level, and SES.
Descriptive statistics were used to summarize the population characteristics (means, standard deviations for continuous data, frequencies for categorical data).
Age, educational level, SES, and nutritional status were categorized for analysis.
Scatterplots were created to visualize BMI trends across age groups.
* BMI trends were calculated using three-year median values. Quadratic polynomial curves were then used to estimate the age at which BMI reaches its maximum value (maxBMI) for the general population and stratified by sex and SES.

* **Software** Data analysis was performed using Python version 3.11.4 and Stata version 17.

## Code

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

We utilized the BMI trajectories function to create plots and gather statistics for each population sample without distinguishing by any other variables. The trajectories were determined for the entire population in every dataset and are defined as follows:
      
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

----------------------------------------------------------------------------------------------------------------------------------------------------------
### Notes
The authors keep the results and conclusions of the project secret until the scientific article is published. When the article is published, the results, discussion and conclusions of the project will be published too, along with its DOI.

Thank you for your interest. If you have any more questions, please contact us.
Contact: steffstaurini@gmail.com
