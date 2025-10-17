<h1>MediAct test for Data Science Position</h1>

“Data Wrangling, Machine Learning Mini Task, Logic & Problem Solving”).


<h2>Overview</h2>

The project consists of 3 questions
- Q1: Data Wrangling
- Q2: Machine Learning Mini Task
- Q3: Logic & Problem Solving

<h1>Q1: Data Wrangling</h1>

<h2>Dataset</h2>

Source: https://drive.google.com/file/d/19p85w5O3ET4HMlJvYREUis0WqHOcwRyw/view
- 1000 samples,
- 2 key features:
  - department
  - work_hours
- labels: performance_score
- preprocessing steps:
  - drop duplicates
  - fill missing work_hours with mean
  - drop rows with performance_score outside of the range of 0 - 100

<h2>Results</h2>

Report means of work_hours by department

- ER	  8.99
- ICU	  9.02
- OPD	  9.01
- Ward	8.96

<h1>Q2: Machine Learning Mini Task</h1>
Source: https://drive.google.com/file/d/1A8JVygr_iGHNN6glCfegRE3OrZSOVSXs/view

Description: 
- 2000 samples,
- 3 key features:
  - age
  - experience_years
  - training_hours
- labels: passed_exam (0, 1)
- preprocessing steps: drop duplicates, drop NA

Data Split: e.g., 80% train / 20% test

<h2>Model Training</h2>

Model: LogisticRegression

<h2>Evaluation</h2>

Metric Score

- Accuracy    0.83
- Precision	  0.80
- Recall	    0.84

<h2>Coefficients</h2>

- experience_years      0.4696
- training_hours        0.1971
- age                  -0.0058

<h2>Results</h2>

The baseline LogisticRegression model achieved 83% accuracy.

The variable with the highest impact is experience_years with the highest weight of 0.4696

<h1>Q3: Logic & Problem Solving</h1>

<h2>Logic</h2>

1. Create all possible combinations

   <pre>python<br>doc_list = ['A', 'B', 'C', 'D']<br>
    df3 = pd.DataFrame(
     [{'D1S1': doc1, 'D1S2': doc2,
      'D2S1': doc3, 'D2S2': doc4,
      'D3S1': doc5, 'D3S2': doc6,
      'Day1': [doc1, doc2],
      'Day2': [doc3, doc4],
      'Day3': [doc5, doc6]}
     for doc1, doc2, doc3, doc4, doc5, doc6 in itertools.product(doc_list, repeat=6)])</pre>

3. Remove the combinations where the same doctor works on the same day

   <pre>df3 = df3[
    (df3['D1S1'] != df3['D1S2']) &
    (df3['D2S1'] != df3['D2S2']) &
    (df3['D3S1'] != df3['D3S2'])]</pre>

4. Remove duplicate combination such as (B,A) and (A,B)

   <pre># sort lists of docs e.g. from [B, A] to [A, B]
     cols_to_sort = ['Day1', 'Day2', 'Day3']
     for col in cols_to_sort:
     df3[col] = df3[col].apply(lambda x: tuple(sorted(x)))
     # drop duplicated combinations
     df3 = df3.drop_duplicates(subset=['Day1', 'Day2', 'Day3'])</pre>

5. Filter out doctor with 3-day schedules

<pre># count 3-day combination
for doc in doc_list:
  df3[doc] = df3[['D1S1', 'D1S2', 'D2S1', 'D2S2', 'D3S1', 'D3S2']].apply(
  lambda row: list(row).count(doc), axis=1
  )
# filter out 3-day
  df3 = df3[
    ~(
        (df3['A'] == 3) |
        (df3['B'] == 3) |
        (df3['C'] == 3) |
        (df3['D'] == 3)
    )
]</pre>

<h2>Shift Schedule Posibilites</h2>

The number of all shift schedule posibilites is 114

The First 5 Combinations of Shift Schedule

<pre>| Day1 | Day2 | Day3 |
|(A, B)|(A, B)|(C, D)|
|(A, B)|(A, C)|(B, C)|
|(A, B)|(A, C)|(B, D)|
|(A, B)|(A, C)|(C, D)|
|(A, B)|(A, D)|(B, C)|</pre>

<h2>Choose 2 best shift schdules and explain the choices</h2>

1. Doctors should not work with the same colleague along the schedule

   Filter only solutions that do not have the same pair of doctors for each day
   
<pre>df_final = df3[
  df3.apply(lambda r:
              (r['Day1'] != r['Day2']) and
              (r['Day1'] != r['Day3']) and
              (r['Day2'] != r['Day3']),
              axis=1)
].reset_index(drop=True)</pre>

Pick a random schedule

<pre>import random

random.seed(42)
random_idx_1 = random.randint(0, len(df_final)-1)
random_row_1 = df_final.iloc[random_idx_1, :].tolist()</pre>

2. Doctors should have the same amount of shifts

Keep rows that do NOT contain any value from the random row

<pre>df_final_filtered = df_final[~df_final.apply(lambda row: any(x in random_row_1 for x in row), axis=1)]</pre>

Pick a random schedule
<pre>random_idx_2 = random.randint(0, len(df_final_filtered)-1)
random_row_2 = df_final_filtered.iloc[random_idx_2, :].tolist()</pre>

<h2>Results</h2>

The first shift schedule

- Day 1: ('C', 'D')
- Day 2: ('A', 'B')
- Day 3: ('A', 'D')

The second shift schedule

- Day 1: ('A', 'C')
- Day 2: ('B', 'C')
- Day 3: ('B', 'D')

<h2>Rationales behind these two chosen schedules</h2>

The best schedules should not have the pair of doctors on each day and each doctor should have the same number of shifts.

However, in one schedule, doctors cannot work the same number restricted by the number of shifts available.

The solution is to pick the first schedule for the first month or week and another schedule for another month or week.

In summary, each doctor has 3 shifts from 2 different schedules and they will not meet the same colleague each shift they work.
