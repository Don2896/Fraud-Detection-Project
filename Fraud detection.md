# Fraud Detection in Bank Transactions: A Data Analysis and Machine Learning Approach.


**Background Overview**:

Fraud is the deceptive practice intended to result in financial or personal gain at the expense of another party. In the banking sector, fraud takes many forms, including identity theft, transaction manipulation, account takeovers, and unauthorized payments. As digital transactions increase, so does the sophistication of fraudulent activities, making it crucial for banks to develop effective detection systems to safeguard customers and financial institutions from monetary losses, reputational damage, and regulatory consequences. This project aims to analyze transactional data to identify patterns and anomalies indicative of fraudulent activity. By leveraging data analytics and machine learning techniques, the goal is to characterize fraudulent behavior in bank transactions, enabling counter-fraud agencies to enhance their detection and prevention strategies.

---

**Objective**:

This project will aim to address buisness questions that could be put forward to fraud detection analysts tasked with idenfying potentially fraudulent transactions and using their findings to profile customers at risk of fradulent activity. The business questions are as follows:
- What are the characteristics of fraudulent transactions?
- How can we profile customers at risk of fraudulent activity?
- Are there any identifibale trends within in fraudulent transactions?
- How effective is the outlined process in identifying fraudulent transactions

---

**Data Exlploration and Feature engineering**:

The dataset used in this project consists of real or simulated bank transaction records, providing insights into customer transaction behaviors and potential fraud patterns. It includes key attributes such as transaction amounts, timestamps, account details, locations, and fraud labels. By exploring this data, the project seeks to develop a systematic approach to distinguishing fraudulent transactions from legitimate ones, thereby contributing to more effective fraud detection mechanisms.


The data set is initially explored to understand the types of transatctional information provided (columns) for each transaction (rows). Information was obtained on the number of columns present, the data type of each column and if there were any null values 
<img width="415" alt="image" src="https://github.com/user-attachments/assets/92768906-b637-46bf-8e0a-397e916e5874" />
<img width="452" alt="image" src="https://github.com/user-attachments/assets/d6a6894f-7480-4065-bac6-1a8cda12801a" />


We can see the dataset is comprised of 2512 rows and 16 columns. These columns are as follows:

- TransactionID: Unique alphanumeric identifier for each transaction.
- AccountID: Unique identifier for each account, with multiple transactions per account.
- TransactionAmount: Monetary value of each transaction, ranging from small everyday expenses to larger purchases.
- TransactionDate: Timestamp of each transaction, capturing date and time.
- TransactionType: Categorical field indicating 'Credit' or 'Debit' transactions.
- Location: Geographic location of the transaction, represented by U.S. city names.
- DeviceID: Alphanumeric identifier for devices used to perform the transaction.
- IP Address: IPv4 address associated with the transaction, with occasional changes for some accounts.
- MerchantID: Unique identifier for merchants, showing preferred and outlier merchants for each account.
- AccountBalance: Balance in the account post-transaction, with logical correlations based on transaction type and amount.
- PreviousTransactionDate: Timestamp of the last transaction for the account, aiding in calculating transaction frequency.
- Channel: Channel through which the transaction was performed (e.g., Online, ATM, Branch).
- CustomerAge: Age of the account holder, with logical groupings based on occupation.
- CustomerOccupation: Occupation of the account holder (e.g., Doctor, Engineer, Student, Retired), reflecting income patterns.
- TransactionDuration: Duration of the transaction in seconds, varying by transaction type.
- LoginAttempts: Number of login attempts before the transaction, with higher values indicating potential anomalies.<img width="776" alt="image" src="https://github.com/user-attachments/assets/ea6bcc02-4232-41bb-bd11-59e486af365a" />



From the above, we can see from 2512 trasactions that took palce there were 495 accounts. Transactions took place across 43 different locations and seemed to occur within 3 unique hours of they day. A quick check for any possible relationships or correlations shows strongest correlation between customer age and their account balance. Focusing on TransactionAmount, Transatction duration and Login attempts a few things points can be made:

- TransactionAmount having a large standard deviation (291.95) can be understood due to transaction amounts tending to vary largely. The interquartile range will give a more concise indication of spread for these amounts and therefore allows potential outliers to be identified.
- The standard deviation for Login attempts is very small (0.62), and that can be understood as we would not expect a massive variance of login attempts from the mean value of 1.
- The standard deviation for TransactionDuration is a worry, as you would not expect such a large spread of time taken to login. Because standard deviations are more infleunced by outliers, their indications of spread can be skewed. especially with unsymmetrical data like TransactionDuration. Like Transaction Amount, the interquartile range will be a better measuere.


A test for relationships was then conducted on the numurical columns of the data set to see if there were any immediate relationships present, prior to any detailed analysis. The only noteable correlation was the positive correlation between Account balance and Age, which is ideal as you would expect Older account holders to generally have larger account balances than younger account holders. Some descriptive statistics were also obtained for these columns for additional information.


Before identifying potential transaction patterns over time, some feature engineering on the TransactionDate column to allow the values to datetime datatypes so they can be utilised by pandas. The below code details this process:
<img width="573" alt="image" src="https://github.com/user-attachments/assets/868931cf-32f3-428b-b78a-e303a1f8fe1e" />

Once TransactionDates were made into datetime, hour, day of the week, month and transaction time were taken and put into their own coulmns and their frequencies determined. 4pm was the most frequent transaction hour, Monday was the most frequent transaction day, October was the most frequent transaction date, Fort Worth was the most frequent transaction date, 2023-10-16 was the most frequent transaction date, Branch transactions were the most frequent channel. 

What was the distribution of actual transaction amounts? we would expect smaller values to be transacted more frequently than larger ones. Anything opposing this expectation would instantly signal underlying suspicious activty but it would not defintively confirm fraud has occured. The Distribution histogram below shows transactions for smaller amounts occuring far more frequently than those with larger amounts 

<img width="532" alt="image" src="https://github.com/user-attachments/assets/0f98e738-312f-48e7-bb96-cfd8d24664bd" />

it was also clear that transactions did not show any clear seasonality. The line plot below shows transaction amounts peaked around the end of the year which can be put down to christmas purchases, but in general amounts varied greatly throughout the year
 <img width="762" alt="image" src="https://github.com/user-attachments/assets/9d6e30a8-4ef6-4c0c-8ada-10b16b9b4f3b" />

Additinally, the data showed Fort Worth, Los Angeles and Oklahoma City were the cities with the highest number of transactions that occured. The number of transactions carried out in branches outnumbered the number of transactions carried out at an ATM or online. 
<img width="764" alt="image" src="https://github.com/user-attachments/assets/c3a69bc5-c16b-4164-bbda-abf2bb7c1a44" />

<img width="769" alt="image" src="https://github.com/user-attachments/assets/34dccf10-9277-4af9-85a7-f0fada8191ca" />


---
Fruad Detection 

I believe before a transaction can definitively be identified as fraudulent there will be steps to initially flag suspicious transactions. This will narrows the pool of transactions needed to investigate and increase the chance of acutally finding a fraudulent transaction. For a transaction to be considered suspicious it must read true for one of the following 3 conditions: Suspicious timeframe, Suspicious amount or Suspicious location. 
- A transaction with a suspicious time will be one where the time differnce to the previous transaction is less that 1800 seconds and with a location different to the previous location.
- A transaction with a suspicious location will flag accounts with a number of unique locations greater than an established threshold (95 percintile of unique locations count)
- A transaction with a suspicious transaction amount will be one where the amount is considred an outlier (less than an established lower bound and greater than an established upper bound)

With these pramaeters we identify 2378 out of 2512 transaction that can be deemed suspicious. Some analysis was then condected to see how these suspicious transactions differed to non-suspocious transactions
Key obersevations:
- The number of fraudulent transactions (blue line) is significantly higher than non-fraudulent transactions (orange line) across the given time period.
- Both fraudulent and non-fraudulent transactions show a declining trend over time.
- The gap between fraudulent and non-fraudulent transactions remains consistently large, suggesting that fraud is a major issue during these hours.

Potential inferences from these finding can be that suspicious transactions and potential fraudsters behind them tend to operate. Non suspicious transactions are also much lower in volume, suggesting this period might correspond to lower overall activty from genunie users. 

A line plot of daily transactions for suspicious vs non-suspicious transaction shows no seasonal related pattersn. only that there are more suspicious transactions than non suspicious 
<img width="996" alt="Screenshot 2025-03-27 at 18 48 47" src="https://github.com/user-attachments/assets/a70934a6-4b5a-4f7b-9896-3a6c64350087" />
<img width="991" alt="Screenshot 2025-03-27 at 18 49 36" src="https://github.com/user-attachments/assets/5c2e42c0-436a-448d-9d5f-c55e20542dbf" />


The next step was to use these suspicious transaction parameters to determine which customer accounts can be considered high risk. This will be accounts with the most number of suspicious flagged raised. for example, an account that read true for suspicious amounts and location will have a count of 2 total flags. There were a total of 428 accounts with with suspicious flags. The least total of flags was 3 and the most was 25. With number of flags sorted in descending order there is a significant jump in the total from the 26th to 25th account, hence we conclude these top 25 accounts can be considered to have highet risk for fraud

<img width="557" alt="Screenshot 2025-03-27 at 19 18 09" src="https://github.com/user-attachments/assets/ba78ca50-a38b-4863-999d-d0b7daf64cb3" />



Principal Component Analysis (PCA) was applied to identify underlying patterns in fraudulent transactions based on three key indicators: Suspicious_Amount, Suspicious_Location, and Suspicious_Timeframe. The first principal component (PC1) highlights a trade-off between high transaction amounts and rapid transaction timeframes, suggesting that fraudsters may either make large transactions or act quickly to evade detection. The second component (PC2) is primarily driven by location anomalies, indicating that fraud is often associated with unusual geographic patterns. Lastly, the third component (PC3) captures fraud risks influenced by both high amounts and short timeframes simultaneously, suggesting another distinct fraud pattern. By reducing the dataset into these principal components, fraud detection can be improved through clustering transactions based these principal components allowing for better fraud classification.

<img width="486" alt="image" src="https://github.com/user-attachments/assets/2f53a75b-6b8b-4914-9ef2-4868710c6d2a" />

Because most of the observed variation can be captured in PC1 and PC2 a 2D scatter plot will be best to visualise the data. From the Scatter plot below, shows transaction are clustred into three groups. Cluster 0 (blue), cluster 1 (white) and cluster 2 (red). Cluster 2 transactions (red) are more spread out, with some points positioned at extreme PC1 values, possibly indicating highly suspicious transactions. Cluster 0 transactions (blue) are tightly grouped near (0, -0.5), suggesting a different fraud pattern compared to Cluster 2. we can therefore infer that:
- PC1 captures a major fraud-related pattern (high transaction amount,short transaction timeframes).
- PC2 captures a secondary pattern, possibly related to location changes.
- Cluster 2 could represent medium-risk behavior — not as severe as Cluster 1 but more anomalous than Cluster 0.
- Cluster 0 represents low-risk transactions that only raise time-based flags
- Cluster 1 This cluster likely represents high-risk transactions with multiple red flags — potentially fraud-heavy behavior.



<img width="635" alt="Screenshot 2025-03-30 at 20 54 49" src="https://github.com/user-attachments/assets/4a2dea8e-0414-4242-aec3-fea7263dda66" />
<img width="758" alt="image" src="https://github.com/user-attachments/assets/ae4cf864-7a14-4162-adb9-8a7abb9038d2" />






conducting a DBSCAN clustering showed similar scatter pattern to the PCA plot. Since PCA reduces your fraud indicators (e.g., suspicious amount, location, timeframe) into a few components, similar patterns mean that the PCA transformation successfully preserved the key fraud patterns. If the PCA plot shows well-separated groups, and K-Means clusters align with them, it means K-Means is effectively detecting patterns in the data rather than random noise.

<img width="635" alt="image" src="https://github.com/user-attachments/assets/437ab961-82f8-443b-8a0d-c1681ca0e819" />

---

With fraudulent/suspicious transactions already determined using suspicious flags. A fraud risk score was then used to quantify how suspicious each transaction was, there for allwoing me to determine transactions that can be considered high risk. One option being to to identify transactions with the most suspicious flags, but the validity/accuracy of this is limited. Instead I used a fraud risk score based on the Euclidean distance from the origin in PCA space to quantify how anomalous each transaction is, as transactions further from the center represent unusual behavior patterns that are more likely to indicate fraud. The maximum possible fraud score being 1, i determined transactions with fraud scores greater than 0.8 should be considered high risk. The graphic below shows the distribution of fraud fraud risk scores. It is clear that the majority of tranactions have a fraud risk scores that is less than 0.2
<img width="641" alt="image" src="https://github.com/user-attachments/assets/06d57c90-4e83-42a7-a1e2-3161cf9048c5" />

The number of transactions with a fraud score greater than 0.8 was identfied to be 3. These transacions could be considered to have the highest risk of being fraudulent. Each transaction also fell into cluster 2. These tranactions were not in Cluster 1 and also had a total of 1 susspicious flag each, but still had the highest risk scores. They validate the reason for using a fraud risk score as an additional way to determine levels of fraud need for using a fraud risk score, as we would ideally expect  

---

What are the characteristics of fraudulent transactions?

The project explores the key characteristics of potentially fraudulent transactions using engineered features such as Suspicious_Amount, Suspicious_Location, and Suspicious_Timeframe. By applying Principal Component Analysis (PCA), the dataset was reduced into components that highlight underlying patterns in suspicious behavior. The first principal component (PC1) revealed a strong contrast between unusually high transaction amounts and very short timeframes, both of which are frequently linked to fraudulent activity. Cluster analysis further supported these insights by grouping transactions with similar behaviors, helping to pinpoint consistent patterns seen in fraud.


How can we profile customers at risk of fraudulent activity?

High-risk customer profiling was performed by aggregating suspicious activity across individual accounts. Using the sum of binary flags for each suspicious behavior, a Total_Flags score was created for every account. Accounts with high cumulative scores were flagged as potentially high-risk. Additionally, the Fraud Risk Score, calculated as the normalized Euclidean distance from the origin in PCA space, allowed for a scalable, quantitative way to prioritize accounts based on how extreme or anomalous their behavior was in comparison to others. This risk-scoring system helped to identify customers who consistently engage in patterns resembling known fraud profiles.


Are there any identifiable trends within fraudulent transactions?

Yes — the PCA and clustering techniques helped uncover visible trends. PCA visualization showed that fraud-related transactions often cluster in areas with high PC1 values, indicating behaviors like large amounts and quick repeat transactions. The clustering algorithms (K-Means and DBSCAN) revealed that many fraudulent transactions fall into distinct, high-density groups separated from more normal behavior. These clusters were consistent across both unsupervised methods, strengthening the evidence of recurring fraud patterns. Visualizations in both 2D and 3D PCA space made these trends clear and interpretable.


How effective is the outlined process in identifying fraudulent transactions?

The outlined process proved effective in identifying and highlighting transactions with fraud-like characteristics without relying on pre-labeled fraud data. The combination of feature engineering, dimensionality reduction, clustering, and scoring provided a multi-layered approach to detection. High-risk clusters aligned with top-scoring transactions, suggesting that the methods worked cohesively. While not a formal model for classification, the approach functions as a powerful unsupervised anomaly detection pipeline, useful for early detection and triage. Further validation with labeled data could enhance confidence in its predictive capabilities.





---
Conclusion:

This project demonstrates how unsupervised learning techniques, specifically Principal Component Analysis (PCA) combined with clustering algorithms, can be effectively used to detect potentially fraudulent bank transactions in the absence of labeled data. By engineering features such as suspicious transaction amounts, unusual geographic locations, and abnormal timeframes, the analysis was able to highlight behaviors that deviate from the norm.

The application of PCA allowed for dimensionality reduction and clearer visualization of patterns in the data. When clustering was applied to this transformed space, distinct groups of transactions emerged — each with varying levels and types of risk. For instance, Cluster 0 showed minimal red flags, indicating likely legitimate behavior, whereas Clusters 1 and 2 exhibited high rates of suspicious timeframes, locations, and monetary values, suggesting different forms of potentially fraudulent activity.

To quantify and prioritize these risks, a Fraud Risk Score was introduced based on the Euclidean distance from the origin in PCA space. This provided a simple but powerful way to rank transactions by risk and added an interpretable metric for stakeholders to monitor or act upon.

Overall, this approach presents a practical and scalable foundation for fraud detection systems, offering data-driven insights that can enhance existing monitoring efforts. With further tuning and automation, the framework could be extended for real-time applications and continuous fraud prevention strategies.


