# Data-Driven Employee Insights Project

## 📚 Table of Contents
- [Task 1: Sentiment Labeling](#task-1-sentiment-labeling)
- [Task 2: Exploratory Data Analysis (EDA)](#task-2-exploratory-data-analysis-eda)
- [Task 3: Employee Score Calculation](#task-3-employee-score-calculation)
- [Task 4: Employee Ranking](#task-4-employee-ranking)
- [Task 5: Flight Risk Identification](#task-5-flight-risk-identification)
- [Task 6: Predictive Modeling - Sentiment Score](#task-6-predictive-modeling---sentiment-score)

### Key Files
- `full_sentiment_workflow.ipynb`: Full notebook with code and commentary
- `test(in).csv`: Raw dataset
- `test_with_predictions.csv`: Dataset with sentiment results
- `final_report.docx`: Written summary of methodology, results, and insights

## Task 1: Sentiment Labeling

### Summary
This task applies a pretrained NLP model to categorize employee messages as Positive, Neutral, or Negative. The dataset is cleaned, processed, and augmented with sentiment predictions and sentiment scores.

### Outcome
- Total messages labeled: **2,152**
- Labeled columns added:
  - `predicted_label`
  - `sentiment_score`

## 📊 Task 2: Exploratory Data Analysis (EDA)

### 🎯 Objective

The goal of this task is to understand the structure, sentiment distribution, and temporal patterns within the employee messages dataset. These insights will form the foundation for scoring, ranking, and flight risk modeling in subsequent tasks.

### 🛠️ Methodology

1. **Dataset Overview:**
   - Loaded the sentiment-labeled dataset (test_with_predictions.csv) from Task 1.
   - Checked data types, missing values, and overall record count.
   - Verified all messages are non-null and properly dated.

2. **Sentiment Distribution:**
   - Visualized the frequency of each sentiment label (Positive, Neutral, Negative).
   - Observed that Neutral dominates, followed by Positive, and few Negative emails.

3. **Time Trend Analysis:**
   - Extracted year and month from the date field.
   - Created monthly sentiment count plots for 2010 and 2011.
   - Identified that positive sentiment grew slightly in 2011, while negative messages declined.

4. **Word Clouds:**
   - Generated word clouds by sentiment category.
   - Found that Positive messages commonly included words like *hope*, *great*, *appreciate*.
   - Negative emails focused more on *issues*, *problems*, *unavailable*.

5. **Message Length Analysis:**
   - Measured the character length of each email.
   - Found that Positive emails tend to be longer, while Negative messages are shorter and direct.

### 📈 Visualizations

The following visualizations were saved in the `visualizations/` folder:

- `sentiment_distributions.png`
- `sentiment_trend_2010.png`
- `sentiment_trend_2011.png`
- `word_cloud_for_positive_emails.png`
- `word_cloud_for_neutral_emails.png`
- `word_cloud_for_negative_emails.png`
- `email_length_distribution_by_sentiment.png`

### 🔍 Key Insights

- Neutral sentiment dominates, suggesting formal or restrained tone in employee communication.
- Positive sentiment increased in 2011, potentially reflecting improved morale.
- Negative sentiment was rare but clearly associated with operational language.
- Sentiment score correlates with message length — longer emails tend to be more positive.

### ✅ Outcome

This EDA provides valuable understanding of employee communication behavior, equipping us with the knowledge to design sentiment-based scoring, ranking, and risk flags in the upcoming tasks.

## 🧮 Task 3: Employee Score Calculation

### 🎯 Objective

To calculate a monthly sentiment score for each employee based on their messages, enabling the analysis of individual sentiment trends over time.

### 🛠️ Methodology

Each employee message was assigned a numerical sentiment score based on the `predicted_label`:

- **Positive** → +1  
- **Neutral** → 0  
- **Negative** → –1

These scores were then grouped by employee and by month, and summed to produce the monthly sentiment score for each employee.

### 🗃️ Grouping and Aggregation Details

- A new column `year_month` was created by converting the `date` column into monthly periods (`YYYY-MM` format).
- Using `groupby(['from', 'year_month'])`, messages were aggregated for each employee by month.
- The sum of sentiment scores was calculated using `.sum()`, resetting to 0 at the start of each new month.
- Final output: a DataFrame with `from`, `year_month`, and `monthly_sentiment_score`.

### 📤 Output

The results were saved as:

- `monthly_sentiment_scores.csv`

Each row in the file represents one employee’s total sentiment score for a given month.

### 📈 Visualizations

One sample employee’s sentiment trend over time was visualized in a line chart, helping demonstrate how sentiment shifts month-to-month.

- `monthly_sentiment_scored_trend_for_sally.beck@enron.com.png`

### 🧠 Key Insights

- Monthly sentiment scores will be used in Task 4 for ranking employees.
- Consistent low scores may indicate disengagement, while high scores suggest ongoing positivity.
- This score lays the foundation for identifying sentiment-based flight risks.

---

## 🧠 Task 4: Employee Ranking

### 🎯 Objective

Generate ranked lists of employees based on their monthly sentiment scores.

### ✅ Requirements

- **Top 3 Positive Employees**: The three employees with the highest sentiment scores each month.
- **Top 3 Negative Employees**: The three employees with the lowest (most negative) sentiment scores each month.
- **Sorting Criteria**:
  - Sorted by sentiment score (descending for positive, ascending for negative).
  - Tiebreaker: Alphabetical order of employee email.
- **Ranking Source**: Rankings are calculated from the monthly sentiment scores produced in Task 3.
- **Presentation**: Rankings are presented both in tabular format and as visualizations.

### 🧮 Methodology

1. **Grouping and Sorting**:
   - Messages are grouped by the `year_month` column.
   - Sentiment scores are sorted accordingly to identify top and bottom performers.

2. **Ranking Logic**:
   - Top 3 positive employees per month: `df.sort_values(...).groupby(...).head(3)`
   - Top 3 negative employees per month: Same logic, sorted in ascending order.
   - An additional column `rank_type` is added to distinguish between "Top Positive" and "Top Negative".

3. **Combining Results**:
   - The two rankings are concatenated and sorted by `year_month`, `rank_type`, and `from`.

### 📊 Visualizations

Two visual approaches were used to display rankings:

- **Bar Plots (Small Multiples)**:
  - Each subplot shows sentiment scores for the top 3 positive and top 3 negative employees for a given month.
  - Useful for analyzing monthly sentiment trends across individuals.
  - `top_positive_and_negative_scores_per_month.png`
  
- **Grouped Bar Plot**:
  - A single plot showing sentiment scores for both positive and negative rankings across all months.
  - Uses different hues (`rank_type`) to compare score types.
  - `top_positive_and_Negative_Email_Senders_per_Month.png`

### 📝 Notes

- This ranking helps HR and management teams spot trends in employee sentiment and potential engagement risks.
- The report includes a brief discussion of how these rankings were determined, including the grouping and sorting methodology above.

## 📌 Task 5: Flight Risk Identification

### 🎯 Objective
Identify employees who are at risk of leaving the company based on the volume of negative emails they send over time.

### 📋 Criteria for Flight Risk
An employee is flagged as a Flight Risk if they have sent 4 or more negative emails within a rolling 30-day window, regardless of sentiment score values.

### 🧮 Methodology

1. **Filter Negative Emails**  
   Emails were filtered where the sentiment label was classified as `"Negative"`.

2. **Group by Sender**  
   Negative emails were grouped by the `'from'` field to analyze individual employee behavior.

3. **Rolling 30-Day Check**  
   For each employee:
   - The email dates were sorted.
   - A two-pointer technique was applied to find any 30-day period in which 4 or more negative emails were sent.

4. **Flag at-Risk Employees**  
   Employees who met the criteria were added to a set and then converted into a list called `flight_risk_list`.

### 📊 Visualizations

- A **bar chart** showing the top 10 employees by negative email count.
- `top_10_employees_by_Negative_Email_count.png`
- A **separate bar chart** displaying the number of negative emails for employees who were flagged as flight risks.
- `negative_emails_count_for_flight_risk_employee.png`

### 🧾 Output
The final list of flight-risk employees is stored in `flight_risk_list`. These employees demonstrate potential disengagement or dissatisfaction and should be considered for follow-up.

### ⚠️ Notes
- The 30-day window is rolling and not constrained by month boundaries.
- This task is critical for HR and management to identify and address early signs of disengagement or retention risk.

✅ **Key Outcome**: A robust detection method that flags high-risk employees based on behavioral patterns in email communication.
and management teams spot trends in employee sentiment and potential engagement risks.
- The report includes a brief discussion of how these rankings were determined, including the grouping and sorting methodology above.

## 📊 Task 6: Predictive Modeling - Sentiment Score

### 🎯 Objective
Develop a linear regression model to analyze sentiment trends and predict sentiment scores using a variety of independent variables that may influence sentiment scores.

### ✅ Requirements
- Select appropriate features from the dataset that may influence sentiment scores  
  *(e.g., message frequency in a month, message length, average message length, word count)*  
- Split the data into training and testing sets to evaluate model performance  
- Develop a linear regression model and validate its effectiveness using suitable metrics  
- Interpret the model results and discuss the significance of the findings  

## 🛠️ Methodology
- **Selected Features**:  
  - `message_length`  
  - `word_count`  
  - `monthly_message_count`
  - `avg_message_length_monthly`  

- **Steps**:
  - Cleaned and filtered dataset
  - Performed 80/20 train-test split
  - Trained a linear regression model
  - Evaluated with R², RMSE, and MAE

## 📈 Model Evaluation
**Training Performance:**
- **R²**: 0.048
- **RMSE**: 0.139
- **MAE**: 0.116

**Test Performance:**
- **R²**: 0.071  
- **RMSE**: 0.131  
- **MAE**: 0.110  

The model suggests that sentiment score is partially explained by the chosen features. However, significant variance remains unexplained, suggesting other latent factors.

## 🔍 Insights
- Linear regression gives baseline understanding but lacks deep predictive power.
- Most likely needs upgrade to complex models (e.g., Random Forest, XGBoost, BERT) for capturing nuanced sentiment.
- Feature engineering using NLP metrics or email structure could enhance results.

### 📊 Visualizations

To evaluate the model’s performance, we utilized two primary visual diagnostics:
- `top_10_employees_by_Negative_Email_count.png`
- `residual_plot.png`

## 📌 Notes
The task helps quantify sentiment trends and demonstrates potential for forecasting employee engagement using message metadata. Evaluation metrics and plots confirm the model's modest performance.

## 🧾 Final Summary

This project successfully applied Natural Language Processing techniques to classify, analyze, and rank employee email communications based on sentiment. Through a structured 6-task pipeline, it:

- Transformed raw email data into a sentiment-enriched dataset
- Uncovered communication trends via visual EDA
- Quantified employee sentiment behavior over time
- Generated rank list of employees based on their monthly sentiment scores
- Flagged potential flight risks based on communication patterns
- Developed a baseline predictive model for forecasting sentiment shifts

This project provides a strong foundation for future work in employee engagement analysis using more complex models and richer textual features.




 




