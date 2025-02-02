# ![Apple_Changsha_RetailTeamMembers_09012021_big jpg slideshow-xlarge_2x](https://github.com/user-attachments/assets/3c0c7fe9-7dff-4ca6-8487-952168345c93)
# OECD Analysis – Using Python to Analyze the Use of R&D Tax Incentives and their Impact in Promoting R&D in OECD Countries

## Project Overview

This project is designed to showcase profieiency in Python, particularly with cleaning and wrangling data, along with demonstrating a statistical analysis and data storytelling. The dataset includes information about countries (and their OECD membership status), various tax rates, R&D incentives and support within countries, and R&D expenditures.

---

### Why I Chose This Project?
- **Hands-on Learning**: Practical experience with real macroeconomic data and advanced economic problem-solving.
- **Practicality**: This analysis appears to be relevant to that work that is done in the OECD, as this analysis was inspired by an economist working in the organization's Centre for Tax Policy and Administration.

## Data Sheets from the .CSV File

The project uses three main sheets:

1. **ISO**: 3-digit country codes and a dummy for OECD membership.
   - `country`: Country name.
   - `code`: ISO code.
   - `OECD`: Indicator for OECD membership.

2. **EATR**: Estimates of the Effective Average Tax Rate (EATR) for R&D and non-R&D investments for different countries for the years 2019 and 2020. These indicators provide an overview of the tax burden on a hypothetical firm making a standardised R&D investment and benefitting from R&D tax incentives versus a comparable non-R&D investment that does not benefit from R&D tax incentives.
   - `code`: ISO Code.
   - `year`: Year.
   - `STR`: Statutory CIT rate.
   - `EATR_rd`: Effective Average Tax Rate for an R&D investment (including R&D tax incentives).
   - `EATR_nrd`: Effective Average Tax Rate for an non-R&D investment (excluding R&D tax incentives).

3. **R&D intensity and government support**: An indicator on R&D intensity by businesses and an indicator for the amount of support governments provide for R&D through direct spending (as opposed to tax breaks).
   - `rd_gdp`: Business R&D expenditure (% GDP).
   - `df_gdp`: Direct government support for R&D, i.e. subsidies or grants given to R&D investment (% GDP).

## Objective and Workflow

Our goal today is to analyze how R&D tax incentives may affect firms’ incentives to invest in R&D within OECD countries, and we will do this by performing an exploratory data analysis including some graphs and basic statistics that can visualize this.

1. To start, we merge the last three tabs of the excel file into one tab to create a more user-friendly ecosystem to perform the analysis. This new tab includes the country codes, years, tax rates, government support data, and OECD membership status.
```python
# Load all sheets from the Excel file
import pandas as pd
import seaborn as sns
import matplotlib.pyplot as plt

# Read all sheets
FILEPATH = 'RAW_data.xlsx'
dataframes = {}

# Read each sheet
iso_df = pd.read_excel(FILEPATH, sheet_name='ISO', engine='calamine')
eatr_df = pd.read_excel(FILEPATH, sheet_name='EATR', engine='calamine')
gov_df = pd.read_excel(FILEPATH, sheet_name='GovSupport', engine='calamine')

# Merge the dataframes
# First merge EATR with ISO data
merged_df = pd.merge(eatr_df, iso_df, left_on='code', right_on='code', how='left')

# Then merge with government support data
final_df = pd.merge(merged_df, gov_df, left_on='code', right_on='ISO', how='left')

# Clean up duplicate columns
final_df = final_df.drop('ISO', axis=1)

print('Shape of final merged dataset:', final_df.shape)
print('\
First few rows of the merged dataset:')
print(final_df.head())
print('\
Summary statistics:')
print(final_df.describe())
```
2. However, we notice that there is some missing data, particularly the data regarding the Business R&D expenditure along with the Direct government support for R&D. This could be due to data not being reported by these countries, different reporting periods, countries not participating in the data collection process, or an issue with the merge. We must check for unmatched countries between the datasets.
```python
# Check for countries in the merged dataset that have missing rd_gdp or df_gdp values
missing_govsupport = final_df[final_df['rd_gdp'].isna() | final_df['df_gdp'].isna()]

# Display the countries with missing values
print('Countries with missing rd_gdp or df_gdp values:')
print(missing_govsupport[['country', 'code', 'rd_gdp', 'df_gdp']])

# Check if these countries exist in the GovSupport dataset
missing_in_govsupport = missing_govsupport['code'].isin(gov_df['ISO'])
print('\
Are these countries present in the GovSupport dataset?')
print(missing_in_govsupport.value_counts())
```
3. An issue with the merge is the only one of these issues that we can immediately control, and it is something that we should verify given that some of the missing data in our newly merged tab is present in the original file. We'll check the original "GovSupport" sheet to see if there's an issue with the merge or the data structure itself.
```python
# Let's examine the original GovSupport sheet
gov_df_original = pd.read_excel(FILEPATH, sheet_name='GovSupport', engine='calamine')
print("Original GovSupport sheet structure:")
print(gov_df_original.head())

# Check the merge keys
print("\
Unique ISO codes in GovSupport sheet:")
print(gov_df_original['ISO'].unique())

print("\
Unique codes in EATR sheet:")
print(eatr_df['code'].unique())

# Check if there's any case mismatch or whitespace issues
print("\
Checking for potential case or whitespace issues in ISO codes:")
print("GovSupport ISO codes with their exact format:")
for code in gov_df_original['ISO'].unique():
    print(f"'{code}'", end=" ")
```
4. After looking into this, we see that there is a mismatch between the country codes among the sheets (as some contain asterisks), so we must clean these ISO codes and merge the data once again into an updated file.
```python
# Clean the ISO codes in GovSupport sheet by removing asterisks
gov_df_clean = gov_df_original.copy()
gov_df_clean['ISO'] = gov_df_clean['ISO'].str.replace('*', '')

# Merge again with clean codes
merged_df = pd.merge(eatr_df, iso_df, left_on='code', right_on='code', how='left')
final_df_clean = pd.merge(merged_df, gov_df_clean, left_on='code', right_on='ISO', how='left')

# Clean up duplicate columns
final_df_clean = final_df_clean.drop('ISO', axis=1)

# Check the previously missing countries
previously_missing = ['CHN', 'ESP', 'ISR', 'USA']
print("Values for previously missing countries:")
print(final_df_clean[final_df_clean['code'].isin(previously_missing)][['country', 'code', 'rd_gdp', 'df_gdp']].head(8))

# Update the entire merged dataset with cleaned ISO codes
final_df_updated = final_df_clean.copy()

# Display the updated dataset's structure and a preview
print("Updated merged dataset structure:")
print(final_df_updated.info())
print("\
Preview of the updated dataset:")
print(final_df_updated.head())
```
5. Now that the data is cleaned, we can perform our exploratory data analysis. We start this by visualizing the top ten OECD countries by R&D investment. We find that Israel leads in R&D investment using 4.28% of its GDP, followed by South Korea (3.63%) and Japan (2.60%). One interesting thing to note is that these countries, despite having high levels of R&D investment, have moderate effective average tax rates with R&D incentives.
```python
# Filter the dataset for OECD member states
# Assuming 'OECD' column indicates membership (1 for members, 0 for non-members)
oecd_data = final_df_updated[final_df_updated['OECD'] == 1]

# Perform exploratory data analysis
import seaborn as sns
import matplotlib.pyplot as plt
```
6. We’ll now observe the top ten countries for R&D tax subsidies (found by calculating the difference between non-R&D and R&D EATR’s), and we do so as a means of clearly seeing which nations provide the most generous incentives for R&D. We find that Slovakia, France, and Portugal top the list.
```python
# Calculate summary statistics for OECD countries
summary_stats = oecd_data.groupby('country')[['EATR_rd', 'EATR_nrd', 'rd_gdp', 'df_gdp']].mean().sort_values('rd_gdp', ascending=False)

print("\
Top 10 OECD countries by R&D investment (% of GDP):")
print(summary_stats[['rd_gdp', 'EATR_rd', 'df_gdp']].head(10))
```
7. Next, we’ll visualize the relationship between R&D tax incentives/subsidies and business expenditure for R&D using scatterplots. The first observes the relationship between the EATR on R&D and the business expenditure for R&D, and the second one shows the tax subsidy (which we’ve calculated in the previous step) and its relation to business expenditure for R&D.
```python
# Scatter plot: EATR_rd (R&D tax incentives) vs rd_gdp (R&D investment as % of GDP)
plt.figure(figsize=(10, 6))
sns.scatterplot(data=oecd_data, x='EATR_rd', y='rd_gdp', hue='country', legend=False)
plt.title('R&D Tax Incentives (EATR_rd) vs R&D Investment (rd_gdp) for OECD Countries')
plt.xlabel('Effective Average Tax Rate on R&D (EATR_rd)')
plt.ylabel('R&D Investment as % of GDP (rd_gdp)')
plt.legend(bbox_to_anchor=(1.05, 1), loc='upper left', title='Country')
plt.grid(True)
plt.show()

# Correlation analysis
correlation = oecd_data[['EATR_rd', 'rd_gdp']].corr()
print("Correlation between EATR_rd and rd_gdp:")
print(correlation)

# Calculate the tax subsidy (difference between non-R&D and R&D EATR)
oecd_data['tax_subsidy'] = oecd_data['EATR_nrd'] - oecd_data['EATR_rd']

# Plot tax subsidy vs R&D investment
plt.figure(figsize=(10, 6))
sns.scatterplot(data=oecd_data, x='tax_subsidy', y='rd_gdp', hue='country', legend=False)
plt.title('R&D Tax Subsidy vs R&D Investment for OECD Countries')
plt.xlabel('Tax Subsidy (EATR_nrd - EATR_rd)')
plt.ylabel('R&D Investment as % of GDP')
plt.legend(bbox_to_anchor=(1.05, 1), loc='upper left', title='Country')
plt.grid(True)
plt.show()

# Calculate average tax subsidy by country
avg_subsidy = oecd_data.groupby('country')['tax_subsidy'].mean().sort_values(ascending=False)
print("\
Top 10 countries by R&D tax subsidy:")
print(avg_subsidy.head(10))
```
 ![OECD1](https://github.com/user-attachments/assets/4fd16bd4-4337-43c5-ad8d-e389d327a9b0)
 ![OECD2](https://github.com/user-attachments/assets/b605cc0b-e0c9-4144-b356-0d7936939d8b)
8. We find a moderate-positive correlation (0.35) between EATR_rd and rd_gdp, which illustatates that lower tax rates are not particularly significant in determining R&D expenditure by businesses. We also see that some countries that offer high R&D incentives do not necessarily experience higher R&D investment by its firms, and that these investment levels can vary among countries that may possess similar incentives. Other factors like political stability, infrastructure, or human capital may be more significant and could be worth exploring in a separate analysis. Despite our main research question being answered, it may be interesting to explore the relationship between the R&D tax incentives and direct government support for R&D. Thus, we will create a scatterplot illustrating the relationship between EATR_rd and rd_gdp. We will also observe the correlation coefficient of these variables.
```python
# Scatter plot: EATR_rd (R&D tax incentives) vs df_gdp (Direct government support for R&D)
plt.figure(figsize=(10, 6))
sns.scatterplot(data=oecd_data, x='EATR_rd', y='df_gdp', hue='country')
plt.title('R&D Tax Incentives (EATR_rd) vs Direct Government Support (df_gdp) for OECD Countries')
plt.xlabel('Effective Average Tax Rate on R&D (EATR_rd)')
plt.ylabel('Direct Government Support for R&D as % of GDP (df_gdp)')
plt.legend(bbox_to_anchor=(1.05, 1), loc='upper left', title='Country')
plt.grid(True)
plt.show()

# Correlation analysis
correlation_df_gdp = oecd_data[['EATR_rd', 'df_gdp']].corr()
print("Correlation between EATR_rd and df_gdp:")
print(correlation_df_gdp)
```
![OECD3](https://github.com/user-attachments/assets/f342e479-b95b-4912-b099-89f2a6b60e28)
We find that the scatterplot along with the correlation coefficient of -0.11 shows us that countries with higher tax incentives for R&D may rely slightly less on direct government support for R&D, but this relationship is rather weak.

To conclude, we have learned that the R&D tax incentives imposed by OECD member states likely have very little effect on their firms’ investments into R&D, which means that there are likely other factors worth analyzing that could have a more significant effect on R&D in these nations.

## Project Focus

This project primarily focuses on developing and showcasing the following SQL skills:

- **Complex Joins and Aggregations**: Demonstrating the ability to perform complex SQL joins and aggregate data meaningfully.
- **Window Functions**: Using advanced window functions for running totals, growth analysis, and time-based queries.
- **Data Segmentation**: Analyzing data across different time frames to gain insights into product performance.
- **Correlation Analysis**: Applying SQL functions to determine relationships between variables, such as product price and warranty claims.
- **Real-World Problem Solving**: Answering business-related questions that reflect real-world scenarios faced by data analysts.


## Dataset

- **Size**: 1 million+ rows of sales data.
- **Period Covered**: The data spans multiple years, allowing for long-term trend analysis.
- **Geographical Coverage**: Sales data from Apple stores across various countries.

## Conclusion

By completing this project, I have developed advanced SQL querying skills, improved my ability to handle large datasets, and gained practical experience in solving complex data analysis problems that are crucial for business decision-making.

---
