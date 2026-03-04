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

    #ultimo 2 oct 25
import numpy as np
import matplotlib.pyplot as plt
import math
from scipy.stats import t

def df_plots(
    df,
    column_name_edad,
    column_name_target,
    titulo='',
    marker='o',
    markersize=10,
    color_median='#121212',
    color_fitted='#6C6C6C',
    min_obs=5,
    xlim=(20, 85),
    ylim=(24, 32)
):
    # Fondo transparente opcional
    plt.rcParams['axes.facecolor'] = 'none'
    plt.rcParams['savefig.facecolor'] = 'none'

    pointx, pointmedian = [], []

    edad_min, edad_max_rango = int(xlim[0]), int(xlim[1])
    for n in range(edad_min, edad_max_rango + 1):
        ventana = df[
            (df[column_name_edad] == n) |
            (df[column_name_edad] == n + 1) |
            (df[column_name_edad] == n + 2)
        ][column_name_target].dropna()

        if len(ventana) >= min_obs:
            pointx.append(n)
            pointmedian.append(float(np.median(ventana)))

    if len(pointx) < 3:
        print(f"[AVISO] Pocos puntos válidos para ajustar en '{titulo}'. "
              f"Se encontraron {len(pointx)} edades con datos.")
        return (np.nan, None)

    plt.figure(figsize=(7, 5))

    # # Puntos (mediana por ventana de 3 años)
    # plt.plot(
    #     pointx, pointmedian,
    #     linestyle='none',
    #     marker=marker,
    #     markersize=markersize,
    #     color=color_median,
    #     label='Median BMI (3y window)'
    # )

    # Ajuste cuadrático
    x = np.asarray(pointx, dtype=float)
    y = np.asarray(pointmedian, dtype=float)
    mask = np.isfinite(x) & np.isfinite(y)

    edad_max = np.nan
    coeffs = None
    if mask.sum() >= 3:
        coeffs, cov = np.polyfit(x[mask], y[mask], 2, cov=True)  # a, b, c
        a, b, c = coeffs
        edad_max = -b / (2 * a) if a != 0 else np.nan
        max_bmi = np.polyval(coeffs, edad_max) if np.isfinite(edad_max) else np.nan

        x_grid = np.linspace(x[mask].min(), x[mask].max(), 300)
        y_grid = np.polyval(coeffs, x_grid)

        # Calculate 95% confidence interval using the covariance matrix
        # This calculation is specific to linear regression, but can be approximated for polynomial fits. It provides an estimate of the uncertainty in the fitted curve.
        X = np.vstack([x[mask]**2, x[mask], np.ones_like(x[mask])]).T
        mse = np.sum((y[mask] - np.polyval(coeffs, x[mask]))**2) / (len(x[mask]) - 3)
        cov_fit = cov # Use this if cov wasn't scaled by residual variance

        # Standard errors of the fitted values on the grid
        X_grid = np.vstack([x_grid**2, x_grid, np.ones_like(x_grid)]).T
        y_err_ci = np.sqrt(np.diag(X_grid @ cov_fit @ X_grid.T))

        # Calculate CI band (using t-distribution for small samples)
        n = len(x[mask])
        t_crit = t.ppf(0.975, n - 3)
        y_ci_lower = y_grid - t_crit * y_err_ci
        y_ci_upper = y_grid + t_crit * y_err_ci

        # Calculate CI for max BMI using delta method approximation
        if np.isfinite(edad_max) and a != 0:
            # Variance of max_bmi = Var(c - b^2/(4a)) approx Var(c) + (b/(2a))^2 Var(b) + (b^2/(4a^2))^2 Var(a) - b/(a) Cov(b,c) - b^2/(2a^2) Cov(a,c) + b^3/(4a^3) Cov(a,b)
            # A simplified approach using the diagonal of the covariance matrix (ignoring covariances between coefficients)
            # This is an approximation and not fully rigorous for the maximum of a polynomial.
            # A more accurate approach would involve the delta method including covariances or bootstrapping.
            # Let's use a simplified version for demonstration, focusing on the uncertainty in the coefficients.
            # A better (but still approximate) variance for the vertex x = -b/2a is var(x) approx (err_b / (2a))^2 + (b * err_a / (2a^2))^2
            # Variance of max_bmi = c - b^2/(4a) approx var(c) + (b/(2a))^2 var(b) + (b^2/(4a^2))^2 var(a)
            err_a, err_b, err_c = np.sqrt(np.diag(cov))
            # Using delta method for variance of max_bmi: f(a,b,c) = c - b^2/(4a)
            # df/da = b^2/(4a^2), df/db = -b/(2a), df/dc = 1
            # Var(f) approx (df/da)^2 Var(a) + (df/db)^2 Var(b) + (df/dc)^2 Var(c) + 2(df/da)(df/db)Cov(a,b) + 2(df/da)(df/dc)Cov(a,c) + 2(df/db)(df/dc)Cov(b,c)
            # This requires the full covariance matrix. Let's use the full delta method for better approximation.
            J = np.array([
                b**2 / (4 * a**2),  # df/da
                -b / (2 * a),       # df/db
                1                   # df/dc
            ])
            var_max_bmi = J.T @ cov @ J
            std_err_max_bmi = np.sqrt(var_max_bmi)
            max_bmi_ci = (max_bmi - t_crit * std_err_max_bmi, max_bmi + t_crit * std_err_max_bmi)

            print(f"[Resumen] Máximo estimado a edad = {edad_max:.2f} años")
            print(f"          IMC en el máximo = {max_bmi:.3f}")
            print(f"          IC95 del IMC en el máximo = [{max_bmi_ci[0]:.3f}, {max_bmi_ci[1]:.3f}] (t_{n-3} = {t_crit:.3f})")

        # Plot CI band
        plt.fill_between(x_grid, y_ci_lower, y_ci_upper, color=color_fitted, alpha=0.2, label='95% CI')

        # Plot the fitted curve
        plt.plot(x_grid, y_grid, color=color_fitted, linewidth=2.0, label='Quadratic fit')
    else:
        print(f"[AVISO] No se pudo ajustar cuadrática en '{titulo}'.")
        print("[Resumen] Máximo estimado a edad = NaN")
        print("          IMC en el máximo = NaN")
        print("          IC95 del IMC en el máximo = [NaN, NaN]")


    # Ejes, títulos y límites
    plt.title(titulo)
    plt.xlabel("Age (years old)")
    plt.ylabel("Median BMI")
    plt.xlim(xlim)
    plt.ylim(ylim)
    plt.xticks([20, 40, 60, 85])
    plt.yticks(list(range(int(ylim[0]), int(ylim[1]) + 1)))

    # #línea roja suave SIEMPRE + texto más arriba ===
    # if np.isfinite(edad_max) and (xlim[0] <= edad_max <= xlim[1]):
    #     soft_red = '#FF3B30'  # rojo suave
    #     plt.axvline(edad_max, color='red', linestyle='dashed', linewidth=1.2, alpha=0.4)
    #     y_top = ylim[1] - 0.03 * (ylim[1] - ylim[0])  # ~3% debajo del borde superior
    #     plt.text(
    #         edad_max, y_top,
    #         f"max @ {edad_max:.2f}y",
    #         rotation=90, va='top', ha='center',
    #         fontsize=9, color='red', alpha=0.75
    #     )

    plt.tight_layout()
    plt.show()

    # Return the calculated values for potential further use
    return {
        'max_age': float(np.round(edad_max, 2)) if np.isfinite(edad_max) else np.nan,
        'y_at_max': float(max_bmi) if np.isfinite(max_bmi) else np.nan,
        'ci95_at_max': max_bmi_ci if 'max_bmi_ci' in locals() else (np.nan, np.nan),
        'coeffs': coeffs,
        'rss': np.sum((y[mask] - np.polyval(coeffs, x[mask]))**2) if coeffs is not None else np.nan,
        'sigma2': mse if 'mse' in locals() else np.nan,
        'dof': n-3 if 'n' in locals() else np.nan,
        'tcrit': t_crit if 't_crit' in locals() else np.nan,
        'n_points': len(pointx)
    }
We utilized the BMI trajectories function to create plots and gather statistics for each population sample without distinguishing by any other variables. The trajectories were determined for the entire population in every dataset and are defined as follows:
      
# generar curvas sin estratificar
df_plots(df_enfr05, 'edad', 'imc', color_fitted='#773ca4')
df_plots(df_enfr09, 'edad', 'imc', color_fitted='#66acff')
df_plots(df_enfr13, 'edad', 'imc', color_fitted='#fc231b')
df_plots(df_enfr18, 'edad', 'imc', color_fitted='#c1971e')

We filtered the function to plot BMI curves by gender for each population.

# curvas por sexo
# enfr 05
df_plots(df_enfr05[(df_enfr05['sexo'] == 1)], 'edad', 'imc', color_fitted='#773ca4')
df_plots(df_enfr05[(df_enfr05['sexo'] == 2)], 'edad', 'imc', color_fitted='#773ca4')
# enfr 09
df_plots(df_enfr09[(df_enfr09['sexo'] == 1)], 'edad', 'imc', color_fitted='#66acff')
df_plots(df_enfr09[(df_enfr09['sexo'] == 2)], 'edad', 'imc', color_fitted='#66acff')
# enfr 13
df_plots(df_enfr13[(df_enfr13['sexo'] == 1)], 'edad', 'imc', color_fitted='#fc231b')
df_plots(df_enfr13[(df_enfr13['sexo'] == 2)], 'edad', 'imc', color_fitted='#fc231b')
# enfr 18
df_plots(df_enfr18[(df_enfr18['sexo'] == 1)], 'edad', 'imc', color_fitted='#c1971e')
df_plots(df_enfr18[(df_enfr18['sexo'] == 2)], 'edad', 'imc', color_fitted='#c1971e')

Then the same procedure was carried out, but this time taking into consideration the socio-economic stratum (SES).

# Trends by SES
# enfr 05
df_plots(df_enfr05[(df_enfr05['nes_cat3'] == 1)], 'edad', 'imc',color_fitted='#773ca4') # low SES
df_plots(df_enfr05[(df_enfr05['nes_cat3'] == 2)], 'edad', 'imc',color_fitted='#773ca4') # middle SES
df_plots(df_enfr05[(df_enfr05['nes_cat3'] == 3)], 'edad', 'imc',color_fitted='#773ca4') # high SES
# enfr 09
df_plots(df_enfr09[(df_enfr09['nes_cat3'] == 1)], 'edad', 'imc',color_fitted='#66acff')
df_plots(df_enfr09[(df_enfr09['nes_cat3'] == 2)], 'edad', 'imc',color_fitted='#66acff')
df_plots(df_enfr09[(df_enfr09['nes_cat3'] == 3)], 'edad', 'imc',color_fitted='#66acff')
# enfr 13
df_plots(df_enfr13[(df_enfr13['nes_cat3'] == 1)], 'edad', 'imc',color_fitted='#fc231b')
df_plots(df_enfr13[(df_enfr13['nes_cat3'] == 2)], 'edad', 'imc',color_fitted='#fc231b')
df_plots(df_enfr13[(df_enfr13['nes_cat3'] == 3)], 'edad', 'imc',color_fitted='#fc231b')
# enfr 19
df_plots(df_enfr18[(df_enfr18['nes_cat3'] == 1)], 'edad', 'imc',color_fitted='#c1971e')
df_plots(df_enfr18[(df_enfr18['nes_cat3'] == 2)], 'edad', 'imc',color_fitted='#c1971e')
df_plots(df_enfr18[(df_enfr18['nes_cat3'] == 3)], 'edad', 'imc',color_fitted='#c1971e')

Finally, the analysis was expanded to include graphing the BMI curves for each socioeconomic status and sex, resulting in 6 graphs for each population under study. The graphs are presented below.

# por sexo y por ses
# ENFR 2005 - Sexo 1 (Male)
print('Male: low, mid, hig')
df_plots(df_enfr05[(df_enfr05['sexo'] == 1) & (df_enfr05['nes_cat3'] == 1)], 'edad', 'imc', color_fitted='#773ca4')
df_plots(df_enfr05[(df_enfr05['sexo'] == 1) & (df_enfr05['nes_cat3'] == 2)], 'edad', 'imc', color_fitted='#773ca4')
df_plots(df_enfr05[(df_enfr05['sexo'] == 1) & (df_enfr05['nes_cat3'] == 3)], 'edad', 'imc', color_fitted='#773ca4')
# ENFR 2005 - Sexo 2 (Female)
print('Female: low, mid, hig')
df_plots(df_enfr05[(df_enfr05['sexo'] == 2) & (df_enfr05['nes_cat3'] == 1)], 'edad', 'imc', color_fitted='#773ca4')
df_plots(df_enfr05[(df_enfr05['sexo'] == 2) & (df_enfr05['nes_cat3'] == 2)], 'edad', 'imc', color_fitted='#773ca4')
df_plots(df_enfr05[(df_enfr05['sexo'] == 2) & (df_enfr05['nes_cat3'] == 3)], 'edad', 'imc', color_fitted='#773ca4')

# ENFR 2009 - Sexo 1 (Male)
print('Male: low, mid, hig')
df_plots(df_enfr09[(df_enfr09['sexo'] == 1) & (df_enfr09['nes_cat3'] == 1)], 'edad', 'imc', color_fitted='#66acff')
df_plots(df_enfr09[(df_enfr09['sexo'] == 1) & (df_enfr09['nes_cat3'] == 2)], 'edad', 'imc', color_fitted='#66acff')
df_plots(df_enfr09[(df_enfr09['sexo'] == 1) & (df_enfr09['nes_cat3'] == 3)], 'edad', 'imc', color_fitted='#66acff')
# ENFR 2009 - Sexo 2 (Female)
print('Female: low, mid, hig')
df_plots(df_enfr09[(df_enfr09['sexo'] == 2) & (df_enfr09['nes_cat3'] == 1)], 'edad', 'imc', color_fitted='#66acff')
df_plots(df_enfr09[(df_enfr09['sexo'] == 2) & (df_enfr09['nes_cat3'] == 2)], 'edad', 'imc', color_fitted='#66acff')
df_plots(df_enfr09[(df_enfr09['sexo'] == 2) & (df_enfr09['nes_cat3'] == 3)], 'edad', 'imc', color_fitted='#66acff')

# ENFR 2013 - Sexo 1 (Male)
print('Male: low, mid, hig')
df_plots(df_enfr13[(df_enfr13['sexo'] == 1) & (df_enfr13['nes_cat3'] == 1)], 'edad', 'imc', color_fitted='#fc231b')
df_plots(df_enfr13[(df_enfr13['sexo'] == 1) & (df_enfr13['nes_cat3'] == 2)], 'edad', 'imc', color_fitted='#fc231b')
df_plots(df_enfr13[(df_enfr13['sexo'] == 1) & (df_enfr13['nes_cat3'] == 3)], 'edad', 'imc', color_fitted='#fc231b')
# ENFR 2013 - Sexo 2 (Female)
print('Female: low, mid, hig')
df_plots(df_enfr13[(df_enfr13['sexo'] == 2) & (df_enfr13['nes_cat3'] == 1)], 'edad', 'imc', color_fitted='#fc231b')
df_plots(df_enfr13[(df_enfr13['sexo'] == 2) & (df_enfr13['nes_cat3'] == 2)], 'edad', 'imc', color_fitted='#fc231b')
df_plots(df_enfr13[(df_enfr13['sexo'] == 2) & (df_enfr13['nes_cat3'] == 3)], 'edad', 'imc', color_fitted='#fc231b')

# ENFR 2018 - Sexo 1 (Male)
print('Male: low, mid, hig')
df_plots(df_enfr18[(df_enfr18['sexo'] == 1) & (df_enfr18['nes_cat3'] == 1)], 'edad', 'imc', color_fitted='#c1971e')
df_plots(df_enfr18[(df_enfr18['sexo'] == 1) & (df_enfr18['nes_cat3'] == 2)], 'edad', 'imc', color_fitted='#c1971e')
df_plots(df_enfr18[(df_enfr18['sexo'] == 1) & (df_enfr18['nes_cat3'] == 3)], 'edad', 'imc', color_fitted='#c1971e')
# ENFR 2018 - Sexo 2 (Female)
print('Female: low, mid, hig')
df_plots(df_enfr18[(df_enfr18['sexo'] == 2) & (df_enfr18['nes_cat3'] == 1)], 'edad', 'imc', color_fitted='#c1971e')
df_plots(df_enfr18[(df_enfr18['sexo'] == 2) & (df_enfr18['nes_cat3'] == 2)], 'edad', 'imc', color_fitted='#c1971e')
df_plots(df_enfr18[(df_enfr18['sexo'] == 2) & (df_enfr18['nes_cat3'] == 3)], 'edad', 'imc', color_fitted='#c1971e')


By Diabetes and Hypertension
# by dbt and no-dbt
# ENFR 2005 - dbt = 0
print('ENFR 2005: dbt 0 y 1')
df_plots(df_enfr05[(df_enfr05['dbt'] == 0)], 'edad', 'imc', color_fitted='#773ca4')
df_plots(df_enfr05[(df_enfr05['dbt'] == 1)], 'edad', 'imc', color_fitted='#773ca4')

# ENFR 2009 - dbt
print('ENFR 2009: dbt 0 y 1')
df_plots(df_enfr09[(df_enfr09['dbt'] == 0)], 'edad', 'imc', color_fitted='#66acff')
df_plots(df_enfr09[(df_enfr09['dbt'] == 1)], 'edad', 'imc', color_fitted='#66acff')

# ENFR 2013 - dbt
print('ENFR 2013: dbt 0 y 1')
df_plots(df_enfr13[(df_enfr13['dbt'] == 0)], 'edad', 'imc', color_fitted='#fc231b')
df_plots(df_enfr13[(df_enfr13['dbt'] == 1)], 'edad', 'imc', color_fitted='#fc231b')

# ENFR 2018 - dbt
print('ENFR 2018: dbt 0 y 1')
df_plots(df_enfr18[(df_enfr18['dbt'] == 0)], 'edad', 'imc', color_fitted='#c1971e')
df_plots(df_enfr18[(df_enfr18['dbt'] == 1)], 'edad', 'imc', color_fitted='#c1971e')

# By hta and non-hta
# ENFR 2005 - hta = 0
print('ENFR 2005: hta 0 y 1')
df_plots(df_enfr05[(df_enfr05['hta'] == 0)], 'edad', 'imc', color_fitted='#773ca4')
df_plots(df_enfr05[(df_enfr05['hta'] == 1)], 'edad', 'imc', color_fitted='#773ca4')

# ENFR 2009 - hta
print('ENFR 2009: hta 0 y 1')
df_plots(df_enfr09[(df_enfr09['hta'] == 0)], 'edad', 'imc', color_fitted='#66acff')
df_plots(df_enfr09[(df_enfr09['hta'] == 1)], 'edad', 'imc', color_fitted='#66acff')

# ENFR 2013 - hta
print('ENFR 2013: hta 0 y 1')
df_plots(df_enfr13[(df_enfr13['hta'] == 0)], 'edad', 'imc', color_fitted='#fc231b')
df_plots(df_enfr13[(df_enfr13['hta'] == 1)], 'edad', 'imc', color_fitted='#fc231b')

# ENFR 2018 - hta
print('ENFR 2018: hta 0 y 1')
df_plots(df_enfr18[(df_enfr18['hta'] == 0)], 'edad', 'imc', color_fitted='#c1971e')
df_plots(df_enfr18[(df_enfr18['hta'] == 1)], 'edad', 'imc', color_fitted='#c1971e')

----------------------------------------------------------------------------------------------------------------------------------------------------------
### Notes
The authors keep the results and conclusions of the project secret until the scientific article is published. When the article is published, the results, discussion and conclusions of the project will be published too, along with its DOI.

Thank you for your interest. If you have any more questions, please contact us.
Contact: steffstaurini@gmail.com
