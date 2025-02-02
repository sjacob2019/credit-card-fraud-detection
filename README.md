# CS 7641 Team 1 - Credit Card Fraud Detection
Joshua Hsu, Shaun Jacob, Andrew Novokshanov, Wanli Qian, Tongdi Zhou

# 1 Introduction 

Every year the recorded number of fraudulent incidents increases worldwide. In 2020, the Federal Trade Commission reported nearly 2.2 million such cases in the United States alone [^fn1]. As technology continues to develop at blistering speeds, so do the ways that criminals steal information. As a result, the technology we use to deal with these problems must also evolve. The goal of our project is to distinguish incidents of fraudulent credit card use from those that are legitimate.

<img src="https://www.ftc.gov/sites/default/files/u544718/explore-data-consumer-sentinel-jan-2021.jpg" width="500"/>

*Image courtesy of https://www.ftc.gov/reports/consumer-sentinel-network-data-book-2020*


# 2 Problem Definition

By taking advantage of Machine Learning we can develop models to determine the legitimacy of a transaction. 

The primary premise of our project is to create Machine Learning models that can accurately predict whether a transaction is fraudulent or legitimate. In other words, when a new credit card transaction is attempted, we will be able to determine with a high degree of certainty whether the transaction should be labeled as fraudulent. 

We will be analyzing the problem at hand using supervised and unsupervised learning methods. Both supervised and unsupervised methods can contribute to our problem analysis in unique ways, especially with respect to the high level of imbalance our dataset exhibits. 

Supervised learning models are commonly attributed to working well on classification problems. However, supervised learning requires labels, and we do not always have labels provided to us in a real-world scenario. In this circumstance, we have decided to also use unsupervised learning models. Unsupervised models do not require labels, and although unsupervised learning models are not commonly used for classification problems, they are useful for determining clustering, association, and performing dimensionality reduction. Unsupervised learning will help us determine what factors are relevant when analyzing the features of our data. 

Supervised learning models do have a limitation though – they are typically more susceptible to outliers and data imbalances, which is a prominent issue in our data. To alleviate the class imbalance, we propose a combination of the under-sampling technique as well as the SMOTE algorithm to decrease the amount of data in the majority class (legitimate transactions) and increase the amount of data in the minority class (fraudulent transactions). Additionally, for certain supervised learning methods we will also utilize class weights which are used to combat data imbalance as they allow the classifier to value minority classes more heavily to the point where the imbalance does not matter. 

Through a combination of these learning techniques, we hope to develop a holistic analysis of how various machine learning algorithms handle the classification of the legitimacy of credit card transactions.

# 3 Data Collection

Fortunately, the problem we are exploring has been researched before and there are datasets available for us to use online. We ultimately decided to work with a dataset found on [Kaggle](https://www.kaggle.com/mlg-ulb/creditcardfraud), since it is composed of real-life data samples and has 284,807 transactions listed. These transactions came from a period of two days by European card holders in September 2013. 

Although the data being grouped by a similar time range helps us analyze similar data, it also means that we have severely unbalanced data. Of the 284,807 transactions in the dataset, only 492 are fraud (0.173%). 

Moreover, due to the sensitive nature of using real life transaction data, much of the identifying data had to be anonymized. Most of the original features were not provided, and instead we were given 28 principal components obtained using PCA, the Transaction Time, the Amount. We were also provided the original ground truth labels for the dataset to use for evaluating the model and for supervised learning.

# 4 Data Visualization and Preprocessing

An important step in machine learning is data visualization, which provides us with an accessible way to better understand the characteristics, trends, and patterns of the dataset. In this section, we will visualize our data, both before and after preprocessing. The figure below shows the scatter plot of the three PCA components with the highest variances. We can see the dataset has a significant imbalance with the genuine class overwhelming the fraud class. We also notice a large overlap between the two classes of data, which brings challenges to the classification tasks. 

<img src="./Images-MidTerm/Preprocess/Preprocess1.jpeg" alt="Preprocess Figure 1" width="500"/>

One way to combat class imbalance is undersampling the majority class. We under-sample the genuine class by randomly retaining 10% of the genuine transactions. The resulting scatter plot is shown in the figure below. A drawback of under-sampling is some information about the majority class may be lost. As a result, the under-sampling ratio cannot be too low. 

<img src="./Images-MidTerm/Preprocess/Preprocess2.jpeg" alt="Preprocess Figure 2" width="500"/>

To further alleviate class imbalance, we use Synthetic Minority Over-sampling Technique (SMOTE) to increase the number of samples in the minority class [^fn5]. SMOTE synthesizes samples feature-by-feature: for each feature in an original sample, SMOTE finds its k-nearest neighbors in the feature space, chooses one neighbor at random, and picks a point at random along the line segment between the sample and its neighbor as the synthesized new sample. The figure below shows the result of SMOTE where we generate nine new samples from each original sample using its 10-nearest neighbors. After using both the undersampling and the SMOTE, the ratio between the legitimate class and the fraud class decreases from 579:1 to 5.79:1. We will mainly use these two class rebalancing techniques for supervised learning methods since they are more suspectable to an imbalanced dataset.

<img src="./Images-MidTerm/Preprocess/Preprocess3.jpeg" alt="Preprocess Figure 3" width="500"/>

The figure below shows the cumulative total variance of the PCA components for the original data and the data generated by SMOTE. As we can see, for both sets of data, the first 7 PCA components capture over 50% of the total variance, while the first 16 PCA components capture over 80% of the total variance. This information can guide the number of PCA components required for training classification models. We expect the first 16 PCA components to contribute the most to our models and the rest of the PCA components may only bring negligible improvement. 

<img src="./Images-MidTerm/Preprocess/Preprocess4.png" alt="Preprocess Figure 4" width="700"/>

The next figure illustrates the scatterplot matrix showing the relationship between the first 7 PCA components, where the red dots represent fraudulent transactions and the blue dots are genuine transactions.  We can see that although no two features can perfectly separate the two classes, each combination of the features can separate some parts of the data, which will be exploited by our models to obtain better classification performance. We also observe that the boundaries between the two classes are highly nonlinear. As a result, we expect machine learning techniques that produce nonlinear decision boundaries to have better performance in classifying the two classes.  

<img src="./Images-MidTerm/Preprocess/Preprocess5.png" alt="Preprocess Figure 5" width="900"/>




# 5 Unsupervised Learning Methods

For our particular dataset, unsupervised learning becomes particularly useful. Because of confidentiality issues, most of the original features are not provided. Our dataset only contains the outputs of a PCA transformation. We experimented with various unsupervised learning methods, including probabilistic clustering algorithms such as GMM and deterministic clustering algorithms like K-Means and DBSCAN. By comparing these three methods, we will try to look at which method yields better clustering performance for our dataset. Since we are using clustering algorithms, we chose to omit the Time feature from our dataset due to the fact that we are treating each data point as a discrete transaction. Therefore, time would not play a significant role in differentiating the 2 classes.

## 5.1 K-Means

### Motivation

We start with the K-Means algorithm since it is one of the easiest clustering algorithms to implement. Although the problem is NP-hard, efficient heuristic algorithms converge quickly to a solution that is good enough for most applications.

### Setup
The K-Means algorithm is initialized by randomly choosing k vectors as the mean vectors for the k clusters. Each data point is then assigned to the k clusters according to their Euclidean distance to the k mean vectors. After the assignment, the k mean vectors are updated using the average of the data points that were assigned to each cluster. The process is iterated until the k mean vectors converge to a steady-state. Since the Euclidean distance is used to assign data points, the decision boundaries for the assignment are linear. As a result, we do not expect good clustering results for our data, since as previous visualizations show, the boundary between the fraud and non-fraud classes is highly nonlinear. For our dataset, we evaluate the clustering performance of the k-means algorithm using all the features available with k = 2.

## 5.2 GMM

### Motivation
When exploring clustering with unsupervised learning methods, GMM is one of the first options chosen. While K-Means suffers due to disregarding variance, GMM uses the covariance of a distribution to cluster data. Furthermore, GMM provides us with soft-clusterings as opposed to K-Means's hard clusterings. This means that should we choose so, we can analyze the likelihood of a given point belonging to a specific cluster.

### Setup
Since the data we were provided was already transformed via PCA, it allows us to skip most of the pre-processing that would be associated with GMM (PCA).

We begin by removing the labels from our dataset and shuffling the data to eliminate any pre-existing bias in the ordering of the data points and to try to ensure an even distribution of fraudulent transactions.

Starting with the first 3 features, we run SciKit-Learn's GMM to group datapoints into two clusters. Next, we need to assign labels to the two clusters. Since this is a binary classification problem, there are two possible label assignments, and we choose the assignment that maximizes the sum of the specificity rate and the recall rate.

After obtaining predictions, we create a confusion matrix of our predictions compared to the data's original labels. Using the confusion matrix, we calculate the recall and specificity of the GMM algorithm.

To examine the impact that increasing features had on the results, we then perform the same operation with additional features, adding a new feature each iteration until all features are used.

To gain a better understanding of the overall similarities between the ground truth and our predicted data, we calculate the Fowlkes-Mallows score, which is the Geometric Mean of the Precision and Recall of our model. We also use the Balanced Accuracy, which is the average of the recall and the specificity. The Balanced Accuracy is especially useful for evaluating the model performance when the dataset is imbalanced.

Finally, to determine how similar and distinct our two clusters are, we calculate the Silhouette score for our newly labelled data.

To better evaluate our model performance and possibly detect overfitting, we also repeat the above process using a K-Fold validation with K = 5. We will then average the Specificity, Recall, and Balanced Accuracy across all folds. 

## 5.3 DBSCAN

### Motivation
The next clustering algorithm we want to examine is DBSCAN due to its relative simplicity compared to GMM and for its ability to handle outliers well. The objective we want to achieve with the use of DBSCAN is to see if there is a clear divide in the data in terms of whether a transaction is fraud or genuine. Additionally, with DBSCAN, we also hope to figure out which features will help the most in terms of clustering.

### Setup
One technique we use to deal with imbalance within the dataset is to undersample the genuine cases by a factor of 10. This still results in around 28,000 cases of genuine transactions and only 492 cases of fraud. Undersampling does not affect the clustering in a significant manner, but due to the random sampling from the under-sampling step and the sensitivity of the DBSCAN algorithm to its parameters, there is sometimes a small discrepancy in the clustering results.

Another benefit of the under-sampling step for DBSCAN is that it cuts down computation time for the algorithm quite heavily since there are ~90% less datapoints which results in about 10x the computation speed. Due to the sensitivity of the DBSCAN algorithm to parameters, this is very important since tuning the parameters often takes 30+ trials to reach optimal parameters. 

Due to the way DBSCAN functions, often times there were a large number of clusters, for example in the 50s, which evidently does not represent the data very well. We made some important interpretations of the clustering by assuming that the first cluster (the 0 cluster) is comprised entirely of legitimate transactions and that any point outside that first cluster is considered a fraudulent case. We interpret the clustering data as such because we expect most of the legitimate transactions to be relatively similar to each other while the fraudulent cases would be comprised of more anomalous cases. For example, for real-life fraudulent transaction detection, often times the more anomalous and unique a transaction, the more likely it can be considered fraud.

# 6 Supervised Learning Methods

Traditionally, supervised learning methods have been used for classification problems. With the knowledge of what label the training dataset has, we can train models that seek out certain features when trying to make a prediction about what label a testing datapoint should be given. A byproduct of supervised learning being more frequently used for classification is that we have many more models to analyze the performance of. Specifically, we chose to focus on K-nearest-neighbors, Random Forests, Support Vector Machines (SVM), Deep Learning, and Logistic regression. In this section we also placed a heavier emphasis on analyzing how SMOTE, Undersampling, and Class Weights can be used to help combat the imbalance in our dataset.

## 6.1 K-nearest-neighbors

### Motivation
The k-nearest neighbors (kNN) algorithm is a non-parametric classification method. It is very easy to implement: for a testing data point, we find its k-nearest neighbors in the training data set and classify the testing data point by a majority vote of the k-nearest neighbors. For example, if out of 10 nearest neighbors in the training data set, 6 are class 0 and 4 are class 1, then the testing data point is assigned label class 0. Here, k is a hyperparameter that needs to be tuned. When k = 1, we simply classify a testing data point as the same class as the training data point that is the closest. Although easy to implement, kNN can be computationally intensive due to the pairwise distance calculation and sorting, especially with a large dataset like ours. 

### Setup
We first split the dataset into training data and testing data by an 80:20 ratio. Next, we perform undersampling as well as SMOTE with varying ratios on the training data. The processed training data will be used for kNN classification where we set k = 5. Finally, we will obtain the classification results on the testing data set.

## 6.2 Random Forest

### Motivation
Random Forest Classification is often praised for not only its accuracy in creating accurate classification models, but also for their efficiency.  Specifically, Random Forest is notably more efficient when run on Large Data sets with higher number of features than its other supervised counterparts such as neural networks. 

With its higher efficiency, we were able to compare how the performance of the Random Forest Classification method was affected as the number of features used in the dataset increased. 

Composed of many decision trees, Random Forests are consistent in outperforming single decision trees for classification problems. This led to the decision to focus our attention on random forests as opposed to decision trees. 

### Setup
We begin our setup by modifying our dataset, specifically separating the labels from the features, and removing the 'Time' feature. The choice to remove the 'Time' feature stems from the fact that in testing, we noted that it consistently harmed our model's accuracy. 

From this modified dataset, we then repeat the following steps for the first 5, 10, 15, 20, 25 features, and finally with all features included. 

We first split the dataset and its labels into a training and testing set. The training set makes up 80% of our total dataset. 

Then, using the SMOTE algorithm, we improve the imbalance between fraudulent and legitimate data in our dataset by generating fraudulent data points until the number of fraudulent datapoints equals 20% the number of legitimate data points in our training data. 

With the adjusted dataset, we then use SKLearn's RandomForestClassifier to train a model on our training data and labels. The hyperparameters used for the RandomForestClassifier included 100 trees, no max depth, and a minimum split of 2 samples.  

With the trained model, we then obtain the model's predictions for our test data. With the predictions and ground truth, we then build the confusion matrices for our results. 

We cross validate the results using K-Folds, with K = 5. 

With the confusion matrices obtained from each of the folds, we proceed to calculate our statistics: the average specificity, recall, balanced accuracy, and precision.  

To ensure a comparison where minimal variables would affect our model, we made sure that our SMOTE algorithm and K-Folds were set to the same random state. This ensured that each training and testing set for each fold was the same across each model, regardless of the number of features. 

Finally, we compare the average statistics we obtained for the models trained on differing number of features.

## 6.3 SVM

### Motivation
Our dataset contains complex data with very high dimensionality. 

SVMs are well known for their effectiveness in high dimensional spaces, where the number of features is greater than the number of observations. The model complexity is of O(n-features * n² samples) so it’s perfect for working with data where the number of features is bigger than the number of samples. 

The SVMS create hyper-planes (could be more than one) of n-dimensional space and the goal is to separate the n-dimensional space.

### Setup
Since our dataset contains highly skewed data, we used random undersampling as well as SMOTE to deal with the skewness. After preprocessing, we split one third of the dataset as the test set and the rest as the training set. The experiment used Sklearn's model selection method to automatically choose the optimal parameters for our model. In the cases of using all of the PCA features, the model selects radial basis function kernel and the C value of 9. Then uses Sklearn's SVM model to perform classification.  

## 6.4 Deep Learning

### Motivation
Neural Networks are one of the most popular machine learning algorithms, and their performance often outperforms other machine learning algorithms, which is one of the reasons we decided to explore this method. A factor that comes into play in the success of our algorithm is domain knowledge, which in traditional machine learning algorithms is used to identify features in order to reduce the complexity of the raw data and make patterns clearer to the algorithm. Another advantage of Neural Networks is that they also work well when there is a general lack of domain knowledge. This is because they learn the high-level features of the dataset in an incremental fashion. Due to that, we don’t have to worry about feature engineering or domain knowledge.  

### Setup
The model architecture was decided through trial and error, and our final architecture is shown in the visualization below. We used a combination of dense and dropout layers to construct our network. The activation function for all of the Dense layers was RELU with the exception of the last layer, which used sigmoid to output the probability that the transaction was fraudulent. The dropout layers were used to randomly drop a fraction of some of the dense layer outputs to mitigate overfitting. 

<img src="./Images-Final/Deep Learning/Model Summary.png" alt="Preprocess Figure 4" width="700"/>

After deciding on the model architecture, we decided to use Adam as our optimizer with a learning rate of 0.001. Since this is a binary classification task and our neural network outputs our prediction as a probability that the transaction is fraudulent, we chose to use binary crossentropy as our loss function. We trained the model with a batch size of 1024 for 11 epochs. The learning rate, batch size, and epochs were all decided via hyperparameter tuning. 

To make the most of our data, we decided to perform K-Fold cross-validation. We split the data into 10 folds, using 9/10 for training/validation and 1/10 for testing. For each K-Fold, we created a model, further split the training data into training and validation splits, trained the model, and evaluated its performance on the testing fold. We averaged the metrics from across all 10 folds to evaluate the general performance of the network.  

To counter the imbalance between fraudulent and legitimate transactions, we decided to try 2 approaches: SMOTE and Class Weights. We used SMOTE to generate extra data points for the fraudulent class so that that Neural Network had more samples to train with. We generated fraudulent transactions to make the ratio between fraudulent and legitimate transactions equal to 0.2. We only performed SMOTE on the training data since we didn't want to modify the validation and testing data. 

The second approach involved us giving each class a custom weight in order to counter the bias already present in the dataset. That is, we made the fraudulent class carry a much higher weight than the legitimate class so that the fraudulent class added a higher penalty to the loss function. This was done in order to influence the neural network to pay more attention to the fraudulent cases.  

## 6.5 Logistic Regression

### Motivation
We chose to implement logistic regression because it is one of the more simple and straightforward supervised learning methods. Additionally, our task deals with a binary classification of whether transactions are genuine or fraudulent, which is the main function of logistic regression.

Logistic regression is a method that computes the probability of a classification using a linear combination of the input features with weights on each feature. The model then utilizes gradient descent in order to tune these weights for each feature in order to improve classification accuracy.

The specific dataset we chose to use has a large imbalance between the number of cases of genuine cases and fraudulent cases where the genuine cases largely outnumber the fraudulent ones. This presents a unique challenge that we hope to solve with various methods such as undersampling, generating additional fraudulent cases through a technique called SMOTE, and using class weights in order to train the logistic regression in order to achieve a more balanced classifier. 

### Setup
To start, we decided to utilize all 28 PCA components from the dataset in order to train our logistic regression model. We decided not to utilize the amount and date features in the original dataset, since they did not significantly affect our results. We then split up the data into training and test sets with a training test split of 80-20. 

After this, we transformed only the training data by utilizing SMOTE, which creates synthetic data points of the minority class (fraudulent) and undersampling of the majority class (genuine) to combat the imbalance in the original dataset. Additionally, we also tested the effects of balanced class weights on the logistic regression classifier which attempts to remedy the class imbalance. The basic premise of using balanced class weights is that the weights for each class are automatically adjusted so they are inversely proportional to the frequency of said class. This causes the classifier to have much higher weight for the minority class, which allows for better classification.

# 7 Results and Discussion

## 7.1 Unsupervised Methods

### 7.1.1 K-Means

<img src="./Images-MidTerm/KMeans/KMeans1.png" alt="KMeans Figure 1" width="700"/>

The figure above shows the result from k-means clustering on the original data with k set to 2. We use all the features for clustering, but we only plot the first three PCA components to visualize the result. Since the k-means algorithm only outputs two clusters with no labels, we need to manually assign labels to the clusters. Since this is a binary classification problem, there are two possible label assignments, and we choose the assignment that maximizes the sum of the specificity rate and the recall rate. As we can see, most data points are clustered to the genuine class, resulting in only 3.6% of the fraud cases being correctly clustered. 

### 7.1.2 GMM

<!-- Non K-Fold Section -->

We started with the more naive approach, where we fitted our GMM to the entire dataset and compared our predictions to the ground truths. For GMM we analyzed and looked at several statistics: the specificity, recall, and Balanced Accuracy of GMM while increasing the number of features. Overall, we were able to obtain high recall, specificity, and Balanced Accuracy, indicating that GMM ended up being useful when trying to cluster datapoints.

<img src="./Images-MidTerm/GMM/TongdiNoKfold.png" alt="GMM No KFold" width="500"/>

As we increased the number of features, the Balanced Accuracy gradually increased as well, indicating better performance as we added more features.

We noticed that specificity steadily decreased until the 20th feature and sharply rose at the 21st feature. PCA outputs the principal components ordered by their variance. This doesn't necessarily imply that that the first few principal components are the most influential features in differentiating the two clusters. We can use the 21st feature as an example. Due to the natural ordering of the PCA components, the 21st feature has a smaller variance than the first few PCA components. Despite this, we experienced a sharp increase in Balanced Accuracy and specificity, indicating that this particular feature had a significant contribution to the model's ability to classify transactions.

<img src="./Images-MidTerm/GMM/GMMNewScores.png" alt="GMM Silhouette Score" width="600"/>

By looking at the silhouette scores and Fowlkes Mallows scores for the clusters, showcasing how different and similar the clusters are to one another respectively, we can further evaluate the cluster assignments of GMM. We consistently see that the silhouette score remains near 0, indicating that the two clusters are not very distinct from one another and that most of the points lie very close to the decision boundary between the 2 clusters. This matches our distribution shown in the preprocessing and visualization section.

Looking at the Fowlkes Mallows score, we can evaluate how closely our predictions matched the ground truth. This score consistently stayed at approximately 0.8, matching what we would expect. Fowlkes Mallows can be used to evaluate the similarity between our true clusters (the labels in our dataset) and the clusters generated by our model's assignment. Since our model's clusterings closely matched the distribution of our dataset and the boundaries between the fradulent and legitimate transactions are not very distinct, we obtain a high Fowlkes Mallows score and a low Silhouette score.


<!-- K Fold Section -->

In an effort to minimize the impact of having such an unbalanced dataset we also used K-Fold cross-validation. The intent of this was to evaluate the performance of GMM given different proportions of fraudulent and legitimate transactions and possibly detect overfitting. The metrics have a higher degree of fluctuation comared to when we did not use K-Fold cross-validation. However, there was only a slight decrease in the overall performance of the model.

<img src="./Images-MidTerm/GMM/TongdiKfold.png" alt="GMM KFold" width="500"/>

We can see why the overall specificity and recall do not differ much from when we do not use K-Fold validation by looking at the following Confusion Matrices. These matrices represent two of the five folds obtained when running GMM on all features.

In these confusion matrices, a label of '0' represents that a data point is legitimate, and '1' represents a data point is fraudulent.

<img src="./Images-MidTerm/GMM/GMMConfusion.png" alt="GMM Matrices" width="700"/>

While four of the folds had results similar to the matrix shown on the left, with high recall and high specificity, there was nearly always a case (shown on the right) with low recall and low specificity. We suspect this is because with five folds, we are spreading the fraudulent testing data too thin, and ultimately end up having a fold where there is poor training data and by extension poor results in testing. When these inaccurate results occur, they drag down the average specificity and recall.

<!-- We can see this by looking at visualizations obtained during two different runs of GMM when looking at the first two features.

<img src="./Images-MidTerm/GMM/GMMVis4.png" alt="GMM Visual Figure 1" width="500"/>
<img src="./Images-MidTerm/GMM/GMMVis1.png" alt="GMM Visual Figure 4" width="500"/>

The most obvious discrepancy we see between some of the visualizations is that sometimes the majority of points are being labeled as 1, and other times as 0. While we can infer from context that most points will be legitimate, and as a result the cluster with more data is legitimate, our data may not always allow for such inferences to be possible and furthermore we might not want to make such guesses.

Looking at actual distribution of each cluster, we also see that there are some pretty clear differences. In cases where many points are being clustered as fraudulent (smaller cluster), we find recall going up at the cost of specificity, and the opposite where we have few data points in the fraudulent cluster. -->



### 7.1.3 DBSCAN

We ended up tuning the epsilon and minimum points per cluster for DBSCAN in order to maximize the Balanced Accuracy. We found that setting the minimum points per cluster to two caused the best results. To tune the epsilon, we set it to an arbitrarily low value and kept increasing it as the Balanced Accuracy increased along with epsilon, until the results "flipped", which was when the true positive rate plummeted to below 50%. 

The following is the confusion matrix for the data after running DBSCAN for all 28 PCA components. The results are normalized in the second figure shown which is based on the ground truth labels so the numbers can be interpreted as percentages. The “0” label represents legitimate transactions, while the “1” label represents fraudulent transactions. With DBSCAN, we are able to achieve a fraud detection rate of around 81%, which represents the percentage of fraudulent cases that are correctly clustered. On the other hand, we achieved a genuine transaction detection rate of over 99% which represents the percentage of legitimate transactions clustered correctly. 

<img src="./Images-MidTerm/DBScan/DBSCANCM.png" alt="DBScan Figure 1" width="400"/>

<img src="./Images-MidTerm/DBScan/DBSCANCMNormalized.png" alt="DBScan Figure 2" width="400"/>

These results are better than expected, since we anticipated that the high-dimensionality of the data would cause issues with the DBSCAN clustering because of the relative simplicity of the algorithm.

Here is the 3-D plot of the clustering of the dataset with DBSCAN which only shows the first 3 PCA components from different perspectives. The different colors correspond to cases of true positives, true negatives, false positives, and false negatives. The true negatives are classified with the color cyan (0), the false positives are the color red (1), the false negatives are the color magenta(2), and the true positives are the color green (3). To reiterate, the positive case, or the "1" case, represents the fraudulent case, while the negative case, or the "0" case, represents the legitimate cases. 

<img src="./Images-MidTerm/DBScan/DBSCANPlot1.png" alt="DBScan Figure 3" width="600"/>

<img src="./Images-MidTerm/DBScan/DBSCANPlot3.png" alt="DBScan Figure 4" width="600"/>

<img src="./Images-MidTerm/DBScan/DBSCANPlot5.png" alt="DBScan Figure 5" width="600"/>

From this plot, we can see that there is a general trend of fraudulent transactions of going in the negative PC1, positive PC2, and negative PC3 direction. Meanwhile, the large bulk of legitimate transactions seem to occur near a zero value for all three principle components. Another observation from the 3-D plot is that nearly all of the false positives trend towards the negative direction in all 3 principle components shown. This shows one of the limitations to our approach to DBSCAN as it failed to identify these trends in the dataset that are easily identified in this plot. 


Our expectations for DBSCAN were not very high when we first experimented with it due to the high-dimensionality of the data, but we also expected DBSCAN to perform better than K-Means due to the ability for DBSCAN to not rely on linear decision boundaries, which we found to be a huge issue for K-Means clustering. While DBSCAN did not perform quite as well as GMM, it was more proficient at properly clustering the genuine transactions with a specificity of over 99%. Overall, DBSCAN proved to be serviceable to clustering the dataset, and serves as an important stepping stone to more robust and accurate supervised classifications that we will explore next. 

## 7.2 Supervised Methods

### 7.2.1 K-Nearest-Neighbors

The figure below shows the kNN performance vs the SMOTE ratio. No undersampling is performed here. The SMOTE ratio is the ratio between the newly generated data and the original data. For example, if the original fraud class has 100 samples, and the SMOTE ratio is 20, then we will generate 2000 fraud samples for a total of 2100 fraud samples. As we can see, the recall as well as the balanced accuracy steadily increase as we increase the SMOTE ratio. This is because the class imbalance is alleviated by the increasing number of fraud data points. We also observe that the specificity is barely affected since no information about the legitimate class is lost. 

<img src="./Images-Final/kNN Results/kNNvsSMOTERatio.jpg" alt="kNN Figure 1" width="600"/>

The figure below shows the kNN performance vs the undersampling ratio. The undersampling ratio refers to the percentage of the genuine data points that are retained. For example, if we have 50,000 data points and we use an undersampling ratio of 0.1, we have 5,000 data points left. As we can see, the recall increases as the undersampling ratio increases since the dataset becomes less imbalanced. However, the specificity decreases. This is because as we remove data points from the legitimate class, we lose information about the legitimate class. As a result, the correctly classified legitimate transactions decrease. Fortunately, the specificity does not decrease significantly when the undersampling ratio is above 0.01.

<img src="./Images-Final/kNN Results/kNNvsUndersamplingRatio.jpg" alt="kNN Figure 2" width="600"/>

When we use both SMOTE and undersampling, we get even better results. With an undersampling ratio of 0.1 and a SMOTE ratio of 50, kNN classification has a specificity of 0.98 and a recall of 0.92, yielding a balanced accuracy of 0.95.  



### 7.2.2 Random Forest

With the default hyperparameters for SKLearn's RandomForestClassifier, we obtained very encouraging results. 

Regardless of the number of features, the models trained using Random Forest all exhibited very high specificity. Unlike with GMM, where we noted that we would oftentimes sacrifice specificity to increase recall by labelling too much data as fraudulent, Random Forest classification was consistent in not mislabeling many legitimate points as fraudulent. 

<img src="./Images-Final/Random Forest/RFResults.jpg" alt="Random Forest Results" width="600"/>

| Category | Value |
|----|----|
|Number of features| 5|
|Specificity| 0.9989905615962277|
|Recall| 0.7188719892192926|
|Balanced Accuracy| 0.8589312754077602|
|Precision| 0.5517431701849772| 
 
For the other statistics we measured however, we did not see incredible results at labelling fraudulent data points correctly at first. When using only 5 features, the recall and balanced accuracy still indicated slightly better performance than if we had labelled fraudulent points randomly.

<img src="./Images-Final/Random Forest/RFConfusion1.jpg" alt="Random Forest Confusion Matrix 1" width="600"/>

^Confusion Matrix for a fold with 5 features^ 

By using fewer features, we gave less opportunities for the random forests to create informative splits that could meaningfully separate fraudulent data from legitimate. We can attribute the specificity of nearly 99.9% and a precision of only 55% to the first 5 features not containing distinguishing factors for fraudulent data.

With 10 features included, we see a drastic improvement in precision and sizeable improvements in recall and balanced accuracy. Although specificity also improved, with the value already greater than 99.8% it makes little difference. As expected, with more features available to the model, there were more opportunities to create more informative splits. Unlike with the results we saw with only 5 features, the inclusion of the additional 5 features must have introduced features that were clearer indicators of whether a transaction was legitimate or fraudulent. 

As we continue increasing the number of features included, we see that the model's specificity, recall, and balanced accuracy increase, although not nearly as greatly as between 5 and 10 features. This implies that while the new features yielded more information, they did not provide much more than the features the model had already been trained on. While precision also tended to increase as the number of features increased, when we went from 25 features to 29 we noticed a slight drop in precision. Although the difference was very small, this may suggest that we had ended up slightly overfitting our model to our training data. 

<img src="./Images-Final/Random Forest/RFConfusion2.jpg" alt="Random Forest Confusion Matrix 2" width="600"/>

^Confusion matrix for a fold with 29 features^ 

Overall, Random Forest yielded very promising results. Unlike with unsupervised learning methods, Random Forest classification was consistent in correctly labelling legitimate data. As we increased the number of features for our training data, we also tended to see an improvement in the model's accuracy both in labelling legitimate data correctly, but also fraudulent. As with most models, if we had more data we would very likely be able to improve our results. 

### 7.2.3 SVM

For experiment with all the PCA features, the confusion matrix is shown below, as well as the metric values.

<img src="./Images-Final/confusionmatrix.png" alt="cm" width="500"/>

Precision: 0.97 
Recall:0.9 
Specificity:0.98 
Balanced Accuracy = 0.94 

Additionally, we tested SVM with different numbers of PCA features. For all of the experiments with different parameters, we found that although all four metrics increased initially, they stabilized when using only the top 15 PCA features. This result is unintuitive compared to other supervised methods. As in some cases, increasing the number of features lead to decreasing metric values. This observation indicates that for SVM, increasing the number of features does not necessarily lead to better results. 

<img src="./Images-Final/metrics.png" alt="m" width="500"/>

To find out how the number of features affects the choice of C value, we plot the C-value against the number of features. We found that the C value plummeted initially, which means that the penalty for misclassification is initially low, but the C value rose back to peak at around 15 features. Later, the C value fluctuates but remains generally stable.  

<img src="./Images-Final/C.png" alt="c" width="500"/>

The ROC(Receiver Operating characteristic) curve is a performance measurement for the classification problem at various threshold settings. ROC is a probability curve and its AUC(area under curve) represents the degree or measure of separability. It tells how capable the model is of distinguishing between classes. The higher the AUC, the better the model is at distinguishing legitimate vs fradulent transactions.  

 <img src="./Images-Final/ROC.png" alt="roc" width="500"/>

### 7.2.4 Deep Learning

#### Using SMOTE 

<img src="./Images-Final/Deep Learning/Training and Validation Metrics SMOTE.png" width="600"/>
<img src="./Images-Final/Deep Learning/Training and Validation Loss SMOTE.png" width="600"/>
<img src="./Images-Final/Deep Learning/Confusion Matrix SMOTE.png" width="600"/>

<ins>Test Metrics</ins>

Specificity: 0.9992719352245331 

Precision: 0.6641232192516326 

Recall: 0.8245526671409606 

Balanced Accuracy: 0.9119123041629791 


Given that around the 7th epoch the validation loss becomes higher than the training loss, it is very likely that the model starts overfitting at the 7th epoch. Another possible indicator of overfitting is the fact that the validation metrics do not follow the same trend as the training metrics and that the training and validation curves are not close to each other. For example, every metric increases every epoch in the training set. However, in the validation set, balanced accuracy and recall start to decrease after the 4th epoch. The overfitting affected the model's test performance. While specificity, balanced accuracy, and recall are all above 82%, the results show an average precision of around 66%. These observations indicate to us that it is very likely that the neural network may be overfitting when we create more fraudulent data points using SMOTE. There are multiple factors that may have caused this, such as the SMOTE ratio, the architecture (number of layers, number of hidden units per layer, activation function), the number of epochs, and the fact that the validation data does not have nearly as much fraudulent cases as the training data. 

#### Using Class Weights

<img src="./Images-Final/Deep Learning/Training and Validation Metrics Class Weights.png" width="600"/>
<img src="./Images-Final/Deep Learning/Training and Validation Loss Class Weights.png" width="600"/>
<img src="./Images-Final/Deep Learning/Confusion Matrix Class Weight.png" width="600"/>

Average Test Metrics 

Specificity: 0.9997432351112365 

Precision: 0.8458663523197174 

Recall: 0.801464968919754 

Balanced Accuracy: 0.9006040871143342 

When using class weights, we get results that are more in line with what we expect for a well performing neural network. Unlike the SMOTE results, the validation metric curves are closer to the training metric curves and they both follow the same general trend. The training and validation loss curves also converge at around epoch 8, which is a good indicator that the model is not overfitting to the training data. The test results show that the model learned better when using class weights. We see a significantly better average precision when using class weights while the other metrics are around the same performance we got from using SMOTE. 

### 7.2.5 Logistic Regression

As a baseline, we ran logistic regression on the original dataset with no transformations on the dataset or class weights and achieved the following:

<img src="./Images-Final/Logistic Regression/logisticregression_original.png" alt="Logistic Regression Figure 1" width="600"/>

This results in: 

Specificity:  0.9998592961288848 

Recall:  0.5619047619047619 

Balanced Accuracy:  0.7808820290168234

#### Undersampling

The undersampling ratio was varied from 1 to 10^4 and the specificity, recall, and balanced accuracy were plotted to see the effects of undersampling on the performance of the classifier. We only undersampled in the training dataset so the test data was untouched.

<img src="./Images-Final/Logistic Regression/logisticregression_US_graph.png" alt="Logistic Regression Figure 2" width="600"/>

As can be seen from the above graph, it seems that the optimal undersampling ratio occurs between 10^2-10^3. Training the logistic regression with an undersampling ratio of 200, we achieved the following results:

<img src="./Images-Final/Logistic Regression/logisticregression_US_CM.png" alt="Logistic Regression Figure 3" width="600"/>

This results in: 

Specificity:  0.9882336387779869 

Recall:  0.8952380952380953 

Balanced Accuracy:  0.9417358670080411

#### SMOTE

For the following graph, the SMOTE ratio is varied from 10-200 and the specificity, recall, and balanced accuracy is plotted to visualize the effects of SMOTE on the performance of the classifier. We only perform SMOTE on the training data, so the test data is completely unaffected.

<img src="./Images-Final/Logistic Regression/logisticregression_SMOTE_graph.png" alt="Logistic Regression Figure 4" width="600"/>

As can be seen, the optimal SMOTE ratio occurs at around 100, which is where the performance of the classifier no longer improves as we increase the SMOTE ratio. The following confusion matrix is generated with a logistic regression classifier with a SMOTE ratio of 100 on the training data. 

<img src="./Images-Final/Logistic Regression/logisticregression_SMOTE_CM.png" alt="Logistic Regression Figure 5" width="600"/>

This results in: 

Specificity:  0.9964648152382293 

Recall:  0.8761904761904762 

Balanced Accuracy:  0.9363276457143528

#### Balanced Class Weights

After training the model using class weights, we evaluate the model on the test data and generate the confusion matrix below.

<img src="./Images-Final/Logistic Regression/logisticregression_weights_CM.png" alt="Logistic Regression Figure 6" width="600"/>

This results in: 

Specificity:  0.9764672775559737 

Recall:  0.9142857142857143 

Balanced Accuracy:  0.9453764959208439

#### Combining Undersampling with Balanced Class Weights

Finally, we combined the undersampling technique with undersample ratio of 200 and balanced class weights and got the following results:

<img src="./Images-Final/Logistic Regression/logisticregression_US_weights_CM.png" alt="Logistic Regression Figure 6" width="600"/>

This results in: 

Specificity:  0.9738466679564521 

Recall:  0.9238095238095239 

Balanced Accuracy:  0.948828095882988

#### Logistic Regression Conclusion

All in all, all of the techniques utilized to combat the imbalanced dataset were quite effective in improving the rate of detection for the fraudulent cases. In terms of pure fraud detection, combining undersampling with class weights was the most effective, but it also led to a noticeable decrease in specificity, which may cause an undesired number of false positives. 

# 8 Conclusion

We found that unsupervised learning yielded fairly good results. However, difficulties arose in determing which clusters corresponded to the fraudulent and legitimate cases.

We found that although the supervised learning techniques far outperformed the unsupervised learning techniques, we had to implement methods to combat the class imbalance to achieve those results. 

Overall, even though supervised learning methods suffer more from imbalanced data, we can use SMOTE, Undersampling, and Class Weights to combat the imbalance and obtain better results with supervised learning than with unsupervised learning.

## Sources:

[^fn1]: Federal Trade Commission. (2021, February). Consumer Sentinel Network Data book 2020. Consumer Sentinel Network. Retrieved October 2, 2021, from https://www.ftc.gov/system/files/documents/reports/consumer-sentinel-network-data-book-2020/csn_annual_data_book_2020.pdf. 

[^fn2]: Hertrich, J., Nguyen, D., Aujol, J., Bernard, D., Berthoumieu, Y., Saadaldin, A., & Steidl, G. (2021). PCA reduced Gaussian mixture models with applications in superresolution. Inverse Problems & Imaging, 0(0), 0. doi:10.3934/ipi.2021053 

[^fn3]: Alqahtani, N. A., & Kalantan, Z. I. (2020). Gaussian Mixture Models Based on Principal Components and Applications. Mathematical Problems in Engineering, 2020, 1-13. doi:10.1155/2020/1202307

[^fn4]: Mahapatra, S. (2019, January 22). Why deep learning over traditional machine learning? Medium. Retrieved October 1, 2021, from https://towardsdatascience.com/why-deep-learning-is-needed-over-traditional-machine-learning-1b6a99177063. 

[^fn5]: Chawla, N. V., Bowyer, K. W., Hall, L. O., & Kegelmeyer, W. P. (2002). SMOTE: synthetic minority over-sampling technique. Journal of artificial intelligence research, 16, 321-357. 
