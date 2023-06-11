# student_behavior_prediction_clickstream_data

https://www.kaggle.com/competitions/edm-cup-2023

## Definition of the prediction task
The prediction task in this project aims at developing a model to predict student performance on educational tasks provided by the ASSISTments online learning platform. The primary objective is to predict, based on a student's clickstream data and problem/assignment-specific data, whether a student will correctly answer a given problem. This is a binary classification problem where the outcome of the two classes, whether the question is correct or incorrect, are “1” and “0”.

Information provided in the data files includes students' actions when doing practices of the platform (such as starting an assignment, responding to a problem, and requesting a hint), curriculum details, assignments details, problems details, hint, and explanation. The ultimate goal of the predictive model is to identify students who may struggle with particular problems, which allows early intervention and targeted assistance to improve academic performance. Meanwhile, the predictive model can significantly contribute to adaptive learning systems, guiding the real-time adjustment of educational content and learning pace based on a student's performance. By identifying areas of struggle or mastery, the learning system can personalize the learning path, either to provide additional resources for students who are not performing well for questions that are difficult for them, or increasing the difficulty level for students who are proficient in specific learning content. This will not only support students who may be not doing well in math, but also optimize the learning experience for each individual. (Muhammad. A, 2016)

## Datasets used in the final submission 

action_logs.csv
The action_logs.csv provides a comprehensive view of student’s behavior when doing the assignments, and their engagement with the platform. For example, it provided timestamp data, which provides an opportunity to build models by students’ behavior overtime. Different types of action might reflect student’s struggles in their learning process, such as multiple attempts in the same questions, whether they have tried to ask for help, etc. 
At the same time, it is a primary file of the data analytics and modeling process in this project. Multiple columns are unique identifiers of assignment, problem, hint, and explanation; which allows it to have connection with other dataset to provide a more detailed context for each student action to improve the predictive accuracy.

assignment_details.csv
It provides detailed information about assignments, including the release and due dates, start and end times, and the related teachers, class, students, and sequence identifiers. 

assignment_relationships.csv
It only has two columns, but it is vital since it serves as a bridge. For example, it can help to understand the connection between student’s performance of different units, and how the behavior changes by assignments.

problem_details.csv 
Each problem’s detail has been provided in this dataset, including problem type, problem formats, related skill represented by code, and BERT embeddings of problem text. By integrating with other datasets such as training_unit_test_scores can get an understanding of how the problem factors affect the grade. 

training_unit_test_scores.csv
This dataset contains information about student scores in unit tests. It is also the training data set for the predictive model that provides a true score. Meanwhile, we can try to connect the student actions and assignment details to the final score by the file to help identify the pattern and relationship of student performance.

evaluation_unit_test_scores.csv
This dataset contains the assignment log IDs that we are required to predict the unit test scores. 

## Data preprocessing and feature engineering

Understanding and preparing the data is a key step in machine learning projects. In this project, we initially performed exploratory data analysis (EDA) to get familiar with what we were working with, followed by data cleaning and preprocessing. We also engineered new features from the existing data to improve model performance.
Data Overview
We began by inspecting the shape of each dataset and checking the missing values. We also wanted to understand the unique values each feature is having. And to get familiar with the dataset, we applied those steps to every data file provided:
action_logs: 23,932,276 rows and 10 columns
assignment_details: 9,319,676 rows and 9 columns
assignment_relationships: 702,887 rows and 2 columns
evaluation_unit_test_scores: 124,455 rows and 4 columns
explanation_details: 4,132 rows and 6 columns
hint_details: 8,381 rows and 7 columns
problem_details: 132,738 rows and 10 columns
sequence_details: 10,774 rows and 8 columns
sequence_relationships: 13,108 rows and 2 columns
training_unit_test_scores: 452,439 rows and 3 columns
To clean the data, we used the “check_missing_values” function to identify the presence of missing values in each dataset. Similarly, we used the “unique_values” function to inspect the unique values of each feature in the datasets.

## Data Cleaning
We started by applying the data cleaning process. Our primary goals were to handle missing values and remove any inconsistencies within the datasets. And through the process of reading through data details in Kaggle and the EDA process, we realize most of the dataset's missing values have meaning, but not really disappeared. We only cleaned the datasets we planned to use for our model, which saved resources and time. Here are some examples of the steps we took:

For the assignment_details dataset, that the assignment_due_date column had missing values. This was because some teachers did not specify a due date. We filled these missing values with 'NA'.

In the problem_details dataset, certain problems were missing because they had been deleted from the database due to errors during their original transcription into ASSISTments. We filled these missing values in the problem_skill_code, problem_contains_image, problem_contains_equation, problem_contains_video, and problem_skill_description columns with 'NA' or 0.

For the sequence_details dataset, some sequences had duplicate rows because they were present in multiple locations within units. Meanwhile, there might be no folder at this path depth. In such cases, for the sequence_folder_path_level_4 and sequence_folder_path_level_5, we filled the column with 'NA'.

## Exploratory Analysis

We only focused on problem_ID because it can directly connect with the testing file, examine the relationship between them and the final score. 
We firstly merged the training unit test scores (tuts) with the problem details (problemd) by the problem_id, to create a new dataframe problem_df for examining the correlation between different variables. For continuous variables, we use the corr() method to calculate correlation of two columns, excluding NA/null values. This gives us a basic understanding of possible relationships between variables.

For exploring categorical variables, we created a plot for the scores distribution based on problem types. We focus on problem_type because of the large number of unique values in problem_skill_code and problem_skill_description. This can reveal patterns in how different problem types might be influencing student performance. Based on the graph, we can see that there might be some relationship between the performance score and problem type, we applied that as one of the predictors. 

## Feature Engineering

We decided the target of feature engineering by providing some assumptions of potential features that have an impact on the predictive result.

Sub - skill involvement (problem_skill_code.csv)
To handle the large number of unique values in “problem_skill_code” of “problem_details.csv”, we created a new binary feature “sub_skill_involved” to indicate whether a sub-skill was involved in the problem. This is determined by checking if the skill code has a dash (“-”), which specifies a more detailed sub-skill.

Dummy Variables (problem_details.csv)
Next, we apply get_dummies() for transforming problem_type features into dummy variables, which is a numerical variable to represent subgroups of the sample in the study. Problem_type is a categorical variable with more than two categories, so dummy variable transformation is needed for model fitting, since we want to apply regression analysis..

After creating these dummy variables, we dropped the original problem_type column from the problemd dataframe, because the original problem_type column is no longer needed after the dummy variables are created.

We then concatenate the dummy variables with the problemd dataframe, which creates a dataframe that each problem type is represented by its own column, with a 1 indicating the presence of the type and a 0 indicating its absence.

Unfinished Assignments (assignment_details.csv and assignment_relationships.csv)
A new binary feature “not finish” is created to indicate whether an assignment was finished or not, determined by whether “assignment_end_time” is null. The data then is merged with “assignment_relationships.csv” by “in_unit_assignment_log_id” as the key.  We aggregated the total number of in-unit assignments(Total_Assignment_Count).Then each unit test assignment’s percentage of unfinished assignment is calculated. Because if the student has difficulty finishing the assignment, it might show the student has a problem in working on that topic. 
forre
Action Logs.csv and Assignment Relationships.csv
We also merged action_logs and assignment_relationships datasets:
We transformed the action column into multiple binary columns with pd.get_dummies, where each column corresponds to a unique action type. This helps to capture the frequency of each action type for each unit test assignment.
We also created a scaled action count (action_count) which represents the total action count normalized between 0 and 1.

Tutoring Availability and Assignment Relationships
We also performed feature engineering using tutoring availability:
We transformed the available_core_tutoring column in a similar way as the action column, wcih provides the frequency of each tutoring type available for each unit test assignment.

Sequence Details, Action Logs, and Assignment Relationships
Finally, we performed feature engineering involving sequence details:
We tried to compute the average number of maximum attempts (average_attempts) for previous in-unit assignments associated with each unit test assignment. This was then scaled using a StandardScaler. However, we found that the average_attemptss do not improve the model performance when trying different combinations of features. So we ultimately removed it.

## Conclusion

Through these feature engineering steps, we hoped to create new features that can capture more complex patterns in the data and improve the performance of our models.

Each of these steps represents a unique session of preprocessing and feature engineering based on different datasets, allowing us to systematically and efficiently transform and enrich our data for model training.

Algorithms/models used to model behavior
After engineering the features and finalizing the testing and training data set, the next step is to apply Machine Learning algorithms to model students’ behaviors. 

Preparing Training and Testing Data Set
We start by merging the features with training dataset (training_unit_test_score) and testing dataset (evaluation_unit_test_scores), where we have selected

Problem details, including problem type, problem skill code, problem type, problem_multipart_position, from “problem_details” dataframe. 
Tutoring availability by available_core_tutoring count features 
Action_count, capture the frequency of each action type for each unit test assignment.
Percentage of unfinished assignment

We then filled all missing values with 0 to handle the missing values. 

Model Training and Evaluation
Before we proceed to model training, we need to specify our input and target columns. Target column is “score”, which is a student's performance on an assignment. The columns that not improving model performances: the unique identifiers “assignment_log_id” and “problem_id”。

We split our training data into a training set and testing set with 80% and 20% split, by using “train_test_split” from the “sklearn.model_selection” library, which we used to evaluate the performance of a model on unseen data. The model training now incorporates the newly created “sub_skill_involved” feature, replacing the previous “problem_skill_code” feature. This shirt in feature selection could possibly adapt the model performance.

Later we initialized a logistic regression model with a maximum interaction of 1000 and fit it on the training data. The target predictive variable is binary, while logistic regression is designed specifically for binary classification problems. Moreover, logistic regression is relatively simple but highly efficient compared with more complex models such as random forests or neural networks. It does not require tuning of many hyperparameters, while it can handle large datasets well. 

After training, we used the model to predict the probabilities of the positive class (score = 1) on the testing set. These predicted probabilities are then used to calculate the area under the receiver operating characteristic (ROC) curve (AUC), which obtained an AUC of approximately 0.70186 after uploading the file to Kaggle. The AUC shows a decent job in this project. 

Results and interpretation 
The logistic regression model achieved an AUC-ROC score of approximately 0.70186, which is relatively promising. It shows that the model has a good capability to predict whether the student can do right and wrong in the unit tests by the learning behaviors captured from the platform. 

Action Counts: 
 The fraction of total actions taken by students in their in-unit assignments can be informative. For example, if a certain action (such as hint requests or incorrect submissions) is overly represented in a student's action log, it could indicate difficulties that might affect their test performance. Educators could intervene by providing additional support or resources to students showing these patterns.

Tutoring Availability: 
The effectiveness of using different types of tutoring on unit tests for prediction, implies that the availability and use of tutoring resources are critical components of online learning success. This could influence decisions about resource allocation and tutoring approach development. Teachers can use the data to determine which types of tutoring are most effective for different kinds of assignments or problems, to fully utilize the more beneficial types of tutoring for students' learning. Meanwhile, if certain types of tutoring methods significantly improve performances, it could be evaluated more heavily for students who are struggling. 

Unfinished Assignments: The significant impact of the percentage of unfinished in-unit assignments is to reflect student’s engagement of learning material, which also shows whether student is struggling with the material. It can prompt teachers to provide additional assistance for labeled students. Also, if certain problems or assignments have too high a finished rate, the teachers might consider reevaluating the difficulty levels with the student's current mastery of the learning section’s practice. This could lead to the development of strategies to increase student engagement and completion rates.

Problem Details: The problem details can provide valuable insights for educational content developers, leading to the creation of more effective learning materials. By observing how students handle sub-skills and multipart problems, we can identify areas of struggle and adaptive learning accordingly. This information can be integrated into a knowledge map, ensuring complex concepts are introduced at the right time.
