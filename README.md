# age-related-BMI-trends
Examining if BMI indicates health decline in low-income groups. Analyzing BMI trends in Argentine adults from 2005 to 2022, using six population surveys. Focus on variations by gender and socioeconomic status


## Introduction
This study focuses on the growing global epidemic of obesity, particularly its impact in Argentina. Body Mass Index (BMI) is used as a tool to measure obesity and assess its link to socio-economic status (SES). Higher BMI is associated with poorer health outcomes and increased healthcare costs. Argentina has seen a rise in BMI, especially among lower SES individuals. Understanding this relationship is crucial for public health strategies.  BMI differences can widen the socio-economic gap and hinder economic development. By analyzing BMI trends by age, sex, and SES, researchers can gain a comprehensive picture of the overweight and obesity epidemic in Argentina. This is especially important considering the country's ongoing economic crisis. The study utilizes data from multiple population-based studies conducted between 2005 and 2022. Researchers hypothesize that BMI trends can reflect not only nutritional status but also health status linked to economic conditions, acting as a proxy for overall well-being. This research aims to estimate how age, sex, and SES affect BMI trends in Córdoba, Argentina (2009-2022) compared to the national population (2005-2018).


## Methods

**Study Design**

This research combines data from two sources:
Córdoba Obesity and Diet Study (CODIES): Cross-sectional surveys conducted in Córdoba, Argentina (CODIES I: 2006, CODIES II: 2022). Both enrolled subjects aged 18-85 years using random samples from the general population.
National Risk Factors Surveys (NRFS): A national survey conducted by the Argentinian government across multiple years (2005, 2009, 2013, 2018). It employed a stratified, multistage approach to collect data from the general population aged 18 or older.

**Data Collection**

CODIES: Data collection involved face-to-face interviews (CODIES I) and phone interviews during the pandemic (CODIES II) with measurements taken post-lockdown.
NRFS: Trained interviewers conducted face-to-face surveys gathering data on household and individual characteristics.

**Measurements**
    
CODIES: Anthropometric measurements (weight and height) were taken following WHO guidelines to calculate BMI.
NRFS: BMI was calculated based on self-reported weight and height, with height and weight being measured in a subsample of the 2018 survey for validation purposes.

**Other Variables**

Age, sex, and socioeconomic status (SES) were included in the analysis.
CODIES: SES used a composite indicator by the National Institute of Statistics and Census (INDEC) considering income, education, and occupation.
NRFS: SES used a composite indicator based on income quintiles and occupation. Both indicators were categorized as low, medium, and high and showed high correlation.

**Statistical Analyses**

This study focused on specific data points: age (continuous and categorical), sex, educational level, and SES.
Descriptive statistics were used to summarize the population characteristics (means, standard deviations for continuous data, frequencies for categorical data).
Age, educational level, SES, and nutritional status were categorized for analysis.
Scatterplots were created to visualize BMI trends across age groups.
BMI trends were calculated using three-year median values. Quadratic polynomial curves were then used to estimate the age at which BMI reaches its maximum value (maxBMI) for the general population and stratified by sex and SES.

**Software**

Data analysis was performed using Python version 3.11.4 and Stata version 17.


**Data source** 

The study utilized data from two sources: the Córdoba Obesity and Diet Study (CODIES) and National Risk Factors Surveys (NRFS). CODIES I and II, conducted in Córdoba in 2006 and 2022, respectively, employed a cross-sectional design with random sampling. CODIES II enrolled 1327 subjects, and both studies covered individuals aged 18-85. NRFS, a government tool for data collection, evaluated risk factors in the Argentine population in 2005, 2009, 2013, and 2018 (n=16,577 for anthropometrics). Trained nutritionists gathered information on demographics, economic status, health, and dietary habits, while utilizing the International Physical Activity Questionnaire to gather data on physical activity. Ethical approval was obtained from the appropriate committees.

**Descriptive statistics** 

analyzed continuous data and BMI trends based on time, gender, age, and SES using Python and Stata. Median moving averages (MMA) were calculated with a lag of age ±1. The first derivative of the quadratic function of age determined the age at which maximum BMI occurred. Statistical analyses were performed with Python v3.11.4, and graphics were created using Stata v17. The analysis involved generating scatterplots of BMI by age and calculating simple moving averages with a three-year width. Median BMI values were used to fit a second-degree polynomial curve, and measures of kurtosis, standard deviation, and skewness were examined. The analyses were carried out on the overall population and then separated by gender and SES (low, medium, high).

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

## Results

**Descriptive results**
The analysis included over 134,000 participants from 2005 to 2022. While there was a slight difference in gender balance between the studies (NRFS with more women), both showed a concentration of adults (30-60 years old) and a similar trend towards intermediate educational levels. Overall average BMI was 26.5, with a slight increase observed over time in both studies. This trend mirrored the rise in obesity prevalence, which jumped from around 16% to nearly 29% in the national survey (NRFS) and from 17% to 26% in the Córdoba-based study (CODIES) during the analyzed period.

### **Main bio-socio characteristics of studied populations: NRFS (2005, 2009, 2013, 2018); CODIES (I and II)**

![image](https://github.com/steffstaurini/age-related-BMI-trends/assets/99434841/e395a47e-9753-4af8-9e26-79d95b4a3927)

**BMI trends across age**
![juntos](https://github.com/steffstaurini/age-related-BMI-trends/assets/99434841/4fbc53d0-1741-4d7b-8ddc-c6a8d0a70b9f)

*Age and BMI Trends*
Figure 2 shows that both national and local BMI trends peak (maxBMI) around 58-59 years old, except for initial data collections (NRFS 2009 and CODIES I) where the peak was slightly later (60-62 years). Notably, the peak BMI itself has also increased slightly across all studies over time.

*Sex and BMI Trends*
Figure 3 explores maxBMI by sex. Women reach their peak BMI (25.9-28.1 kg/m²) between 59 and 70 years old, while men reach theirs (26.4-28.8 kg/m²) between 55 and 60 years old.

*Socioeconomic Status (SES) and BMI Trends*
Figure 4 investigates maxBMI by SES. Lower SES groups tend to have a higher maxBMI (26.4-28.9 kg/m²) reached at younger ages (56-61 years old) compared to higher SES groups. This pattern holds true for both the national survey (NRFS) and the Córdoba study (CODIES).
The middle SES category showed no significant differences between NRFS and CODIES studies. NRFS maxBMI in this group increased steadily over time (26.0 in 2005 to 28.55 in 2018), while the peak age remained constant (58 years old). CODIES data also showed an increase in maxBMI (26.85 to 28.1) but with a significant decrease in the age at which it's reached (from 63 to 53 years old) between CODIES I and CODIES II.
High SES individuals tend to reach their maxBMI later in life and at lower BMI levels compared to lower SES groups. In the NRFS, the maxBMI (24.9 kg/m²) is reached around 60 years old, with a slight increase in peak age and BMI observed in later editions (27.7 kg/m² at 63 years old). CODIES data shows a similar trend, with maxBMI increasing from 25.6 kg/m² at 53 years old in the first edition to 26.8 kg/m² at 57 years old in the second edition.
Overall, this section highlights the interaction between age, sex, SES, and BMI. It reveals that people from lower socioeconomic backgrounds tend to reach a higher peak BMI at younger ages compared to those with higher SES.


## conclusion

The study confirms the intricate link between age, socioeconomic status (SES), and BMI in Argentina, particularly Córdoba.
It identifies clear differences in BMI patterns across SES groups and genders. People with lower SES tend to have higher peak BMIs, highlighting the impact of economic factors on health.
Women reach their peak BMI later than men, indicating gender-specific variations in BMI trajectory.
The study acknowledges the multifaceted nature of poverty as a relevant context for these findings, emphasizing the need for a comprehensive approach to understanding the complex relationship between SES and health.
This research not only supports existing evidence of rising obesity among low-income populations globally, but also emphasizes BMI's potential as a proxy indicator for SES.
These insights are crucial for developing effective public health policies aimed at improving overall health, reducing the burden of non-communicable diseases (NCDs), especially among disadvantaged socioeconomic groups.
Finally, the study underscores the significance of BMI as a valuable tool. Beyond its established role in assessing nutritional status, BMI's potential as a proxy indicator for SES offers a more comprehensive approach to health analysis and informs evidence-based decision-making.

