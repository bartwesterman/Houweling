# Houweling et al | Clustering z-values of Synergies

Written by Anna Giczewska

Part of manuscript "Identification of personalized multi-target therapies against glioblastoma identifies a synergistic interaction between EGFR/HER2/MCL1 and BCL2" Last update: 11/1/2022

## 1.	Introduction
Cluster analysis is a statistical method that attempts to select groups in data such that objects within groups are as similar as possible and simultaneously maximizes the differences between identified groups. Cluster analysis is a commonly used tool for almost a century and many clustering methods were discovered until thus far [1]. The concept of clustering using model-based approach was proposed by Tiedeman in 1955, and thereafter this clustering method has grown into an important subfield of classification [2]. Model based clustering (MBC) assumes that analyzed data is generated by some finite mixture of probability distributions. The one of many notable advantages of MBC over traditional clustering methods is that it separates the data using estimated statistical model and the choice of clustering method turns into a problem of model selection [3]. For identifying best model selection Bayesian Information Criterion (BIC) is commonly used and recommended [4] [5]. Another advantage of MBC is that it identifies the number of clusters in the data, whereas other methods require either a priori assumptions (e.g., k-means) or post hoc subjective decisions (hierarchical clustering) [3].

## 2.	Data preparation
a.	Data preparation: Prior to performing analysis data set was scaled using function scale() in R. 
## 3.	Assess Cluster Tendency:  
a.	Hopkins statistics (H) - is used to assess the clustering tendency of a data set by measuring the probability that a given data set is generated by a uniform data distribution. In other words, it tests the spatial randomness of the data. A value close to 1 tends to indicate the data is highly clustered, random data will tend to result in values around 0.5, and uniformly distributed data will tend to result in values close to 0. 

```
set.seed(123)
get_clust_tendency(hdata, n = nrow(hdata)-1, graph = FALSE)
```

Hopkins statistics in our dataset:
H= 0.6166
This H indicates that data is clusterable with not strong separation between clusters. 


b.	Visual method – PCA. The first component (Dim1) explains 24.7% of the variation, and the second component (Dim2) 12.5%.   

```
fviz_pca_ind(prcomp(hdata, scale = FALSE, center = FALSE), title = "PCA - Data",
             geom = "point", ggtheme = theme_classic())
```

![image](https://user-images.githubusercontent.com/47714729/199244395-feb9fd5f-56bf-49d4-86a7-29ab4bbc8f3e.png)
 
We will use a screen plot to determine how many factors could be used to determine optimal number of factors to be used in PCA.    

```
fviz_eig(prcomp(hdata, scale = FALSE, center = FALSE), addlabels = TRUE)
```
![image](https://user-images.githubusercontent.com/47714729/199244740-d224f185-7723-42ac-bfaf-b7512a2a9550.png)

Through “elbow method” in a scree plot, we can indicated that we could use between 2-4 PCA components. Therefore, we will present our data in PCA with 3 dimensions. 

```
pca_3d = prcomp(hdata, scale = FALSE, center = FALSE)
pca3d(pca_3d)
```
![image](https://user-images.githubusercontent.com/47714729/199244825-cfd99218-dccd-480c-a17f-13ba1b644758.png)

We can see now that our data might actually have some clusters to be distinguished. Since structure of these clusters is not straightforward, we will use model based clustering method as a clustering algorithm. 

## 4.	Model based clustering:  
Since traditional clustering algorithms such as k-means and hierarchical clustering derive clusters directly based on the data rather than incorporating a measure of probability or uncertainty to the cluster assignments. Model-based clustering attempts to address this concern and provide soft assignment where observations have a probability of belonging to each cluster. Moreover, model-based clustering provides the added benefit of automatically identifying the optimal number of clusters. We will perform Gaussian mixture models, which are one of the most popular model-based clustering approaches available (library mclust in R). 
a.	Optimal number of clusters
Key element in model based clustering is a covariance matrix that describes the geometry of the clusters; namely, the volume, shape, and orientation of the clusters. GMMs allow for flexible clustering structures. There are three main families of models: spherical, diagonal, and general. For these families we can identify 14 model types. We can choose multiple methods to identify the best model for our data. Bayesian Information Criterion (BIC) has been shown to work well in model-based clustering. Therefore, we will use BIC to evaluate best model in our analysis. 

```
mc = Mclust(hdata)
summary(mc)
fviz_mclust(mc,"BIC", palette ="jco", stand = FALSE)
```

![image](https://user-images.githubusercontent.com/47714729/199244959-d974e953-7617-472b-892f-118bfd026971.png)

In our analysis, we can observe that EII (spherical, equal volume) model with 3 components was selected as a best model type. 
Note: For some methods, we can see that values are missing (missing dots on the figure above). It is because model could not converge for these parameters.  

b.	Clusters assignment:
EII model:
```
fviz_mclust(mc, "classification", geom =c("text","point"), pointsize =1.5, palette ="jco", stand = FALSE, repel=FALSE) 	
```
![image](https://user-images.githubusercontent.com/47714729/199245081-720547da-3029-4c3e-839d-015eeebde590.png)

```
fviz_mclust(mc, "uncertainty", palette ="jco", stand = FALSE, scale="none")
```
![image](https://user-images.githubusercontent.com/47714729/199245188-d221a999-1cb7-4ba2-bc1f-81e06859e31e.png)

Looking at classification plot and uncertainty plot, we can conclude that group #16 and #43 has the highest uncertainty to where it should be classified. We also can see that elements of cluster 1 : #4 and #16 are looking as misclassification on 2D plot. Therefore, we will plot it in 3D to try to see if there is a difference between these elements.   

```
pca3d(prcomp(hdata, scale = FALSE, center = FALSE), 
      show.labels =T, 
      group=mc$classification,
      show.ellipses = T,
      ellipse.ci = 0.85,
      col = col_vec
)
```		
 
![image](https://user-images.githubusercontent.com/47714729/199245280-c973932d-cb22-4cfe-a32c-abaf0e81fa47.png)
![image](https://user-images.githubusercontent.com/47714729/199245305-1799e7c0-9de7-4ca4-8065-e5eee55b80b4.png)
![image](https://user-images.githubusercontent.com/47714729/199245331-e877d027-4532-4283-8c75-5de68d1aa4eb.png)
![image](https://user-images.githubusercontent.com/47714729/199245380-5fcfe787-e363-403c-af94-69cc9ce12da5.png)
 
On the 3D plot we can see the why these elements were assigned to each cluster.  

VII -  Cluster assignments:	# of elements:	Groups:

| 1. | 11 |  1 |  4 | 14 | 15 | 16 | 26 | 27 | 31 | 32 | 33 | 41 |    |    |    |    |    |    |    |
| 2. | 18 |  2 |  3 |  5 |  6 |  7 |  8 |  9 | 10 | 11 | 13 | 19 | 20 | 22 | 28 | 35 | 37 | 40 | 42 |
| 3. | 14 | 12 | 17 | 18 | 21 | 23 | 24 | 25 | 29 | 30 | 34 | 36 | 38 | 39 | 43 |    |    |    |    |

We can look across our three clusters and assess the probabilities of cluster membership. Plot below presents the distribution of the probabilities for each of 43 observations. 
```
probabilities <- mc$z 
colnames(probabilities) <- paste0('C', 1:3)

probabilities <- probabilities %>%
  as.data.frame() %>%
  mutate(id = row_number()) %>%
  tidyr::gather(cluster, probability, -id)

ggplot(probabilities, aes(probability)) +
  geom_histogram() +
  facet_wrap(~ cluster, nrow = 2)
  ```
  ![image](https://user-images.githubusercontent.com/47714729/199245528-d65dcfbc-4807-4e93-9e9f-b3cc3421ae59.png)

Looking at all clusters, we see that there are not any observations in the middle of the probability range what indicates that separation between clusters is good. C2 has far fewer observations with high probability. This means that C2 is the smallest of the clusters above and that C3 is a more compact cluster. 

## References:
1.	T. Soni Madhulatha, An overview on clustering methods, IOSR Journal of Engineering, Apr. 2012, Vol. 2(4) pp: 719-725
2.	McNicholas, P.D. Model-Based Clustering. J Classif 33, 331–373 (2016). https://doi.org/10.1007/s00357-016-9211-9
3.	John S. Ahlquist, Christian Breunig , Model-based Clustering and Typologies in the Social Sciences; Published online by Cambridge University Press:  04 January 2017, DOI: https://doi.org/10.1093/pan/mpr039
4.	Dasgupta, Abhijit, and Adrian E. Raftery. 1998. Detecting features in spatial point processes with clutter via model-based clustering. Journal of the American Statistical Association 93:294–302
5.	Fraley, Chris, and Adrian E. Raftery. 1998. How many clusters? Which clustering method? Answers via model-based cluster analysis. The Computer Journal 41:578–88.

