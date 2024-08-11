# 🧬 Introduction to Functional Enrichment Analysis  

Single-cell RNA sequencing (scRNA-seq) has transformed our understanding of cellular diversity, allowing researchers to explore cellular heterogeneity with unprecedented detail. To make sense of the vast amount of data generated by scRNA-seq, researchers often turn to three key analytical approaches: Gene Ontology (GO) analysis, pathway analysis, and Gene Set Enrichment Analysis (GSEA). Collectivley, these three methods fall under the broad category of analyses called functional enrichment analysis, which involves various computational techniques used to identify and interpret the biological functions, processes, and pathways that are overrepresented or significant in a given set of genes or proteins. 

In this article, we’ll explore the purpose, benefits, and applications of each approach, using skeletal muscle research as an example. However, it should be noted that all of the analyses discussed in this article have broader relevance in other fields, such as cancer research. Additionally, I’ll show you how to perform each analysis, using real-world scRNA-seq data. 

# 🧬 Gene Ontology (GO) Analysis 
### What Is Gene Ontology Analysis?

Gene Ontology (GO) analysis is a bioinformatics tool that categorizes differentially expressed genes according to their associated biological processes, cellular components, and molecular functions. This categorization is based on a structured, hierarchical vocabulary known as the Gene Ontology, which systematically describes gene functions. In the context of skeletal muscle research, GO analysis provides insights into the biological roles of differentially expressed genes, helping to elucidate the specific effects of exercise interventions.

While differential expression analysis identifies genes that are up- or down-regulated in response to an intervention, treatment, or drug regimen, GO analysis takes this a step further by linking these genes to broader biological contexts. By grouping genes into functional categories, GO analysis can reveal which biological processes, molecular functions, or cellular components are impacted, offering a more detailed understanding of the mechanisms through which an intervention, treatment, or drug exerts its effects.

### How Is GO Analysis Used In Research?

In skeletal muscle research, Gene Ontology (GO) analysis can be used to identify and categorize the biological processes affected by exercise or muscle injury. For example, after resistance training, GO analysis might show an enrichment of genes associated with muscle hypertrophy, such as those involved in "protein synthesis," "muscle fiber development," and "response to mechanical stimulus." This helps researchers understand which specific biological processes are driving muscle growth and adaptation, providing insights that can inform training protocols or therapeutic approaches to enhance muscle repair and regeneration.

### Performing GO Analysis

Before showing you how to perform GO analysis, I’ll import all of the libraries and data used in this tutorial. Notably, the dataset used in this tutorial contains a list of genes that are differential expressed following an exercise stimulus.

```python
# import libraries
import pandas as pd
from gprofiler import GProfiler
import matplotlib.pyplot as plt
import seaborn as sns
import gseapy as gp

# load data and view first few rows
filename = 'skeletal_muscle.csv'
data = pd.read_csv(filename)
skeletal_muscle = pd.DataFrame(data)
skeletal_muscle.head()
```
[INSERT GO1.png]

As you can see in the image above, our data frame contains a column named ```names```, containing the names of genes, a column named ```scores``` which represent a test statistic generated by a Wilcoxon rank-sum test for each gene (A gene with a positive score is up-regulated in the post exercise condition relative to baseline and negative score is down-regulated in the post exercise condition relative to baseline), and two columns named ```pvals``` and ```pvals_adj```, indicating the significance of each gene-associated score.

Now, before performing GO analysis we’l want to view the distribution of our scores, demonstrated in the code block below, to help us determine the optimal filtering criteria for selecting the most significant genes.

```python
# view distribution of scores
sns.set(style="whitegrid")
plt.figure(figsize=(10, 6))
sns.histplot(skeletal_muscle['scores'], bins=50, kde=True)
plt.title('Distribution of Scores from Differential Expression Analysis')
plt.xlabel('Score')
plt.ylabel('Frequency (# of genes)')
plt.show()
```

[INSERT GO2.png]

In the image above you can see that our Wilcoxon rank-sum test scores are normally distributed. We’ll now calacutle the mean and standard deviation of the scores, and then use our mean + 1.5 standard deviations as a threshold for identifying genes that are differentially expressed to a meaningful degree. Then, we’ll filter our genes with scores greater than this threshold and p-values below 0.05 and save these to a variable named ```significant_gene```s before extracting the list of gene names, which will be used in our GO analysis.

```python
# calculate mean and standard deviation of the scores
mean_score = skeletal_muscle['scores'].mean()
std_score = skeletal_muscle['scores'].std()

# set the threshold as mean + 1.5 standard deviations
threshold = mean_score + 1.5 * std_score

# filter for genes with scores greater than the threshold and p-value < 0.05
significant_genes = skeletal_muscle[(skeletal_muscle['scores'] > threshold) & (skeletal_muscle['pvals_adj'] < 0.05)]

# extract list of gene names for these significant genes
gene_list = significant_genes['names'].tolist()
```

Now, we’re ready to perform our GO analysis, as demonstrated in the code block below.

```python
# initialize GProfiler and return result in pandas DataFrame format
gp = GProfiler(return_dataframe=True)

# perform GO analysis using the significant gene list
go_results = gp.profile(organism='hsapiens', query=gene_list)

# display the first few results
print(go_results.head())
```

[INSERT GO3.png]

The output from our GO analysis includes various types of information, as shown in the image above. The ```source``` column indicates the origin of the GO term, where, for example, ```GO:CC``` refers to cellular component terms, and ```GO:BP``` refers to biological process terms. Each GO term has a unique identifier, which is listed in the ```native``` column.

The output also provides biological context for each GO term. The ```name``` column specifies what each GO term represents, and the ```description``` column offers additional details about its biological significance. The statistical significance of the GO terms is indicated by the ```p_value``` column, and the ```significant``` column contains a boolean value (True or False) that shows whether the enrichment is statistically significant.

To understand how certain GO terms are represented in the human genome and in our specific sample, we can look at the ```term_size```, ```query_size```, ```intersection_size```, and ```parents``` columns. ```Term_size``` indicates the total number of genes associated with a specific GO term in the reference human genome, while ```query_size``` shows the number of genes from our gene list that are potentially related to that GO term. ```Intersection_size``` refers to the number of genes from our list that are both associated with the GO term and identified as significant—this is a subset of ```query_size```. The ```parents``` column lists the parent GO terms of the current term, helping us understand the hierarchical relationships between terms.

Finally, the output includes evaluation metrics such as ```precision``` and ```recall```, which provide insight into the quality of the enrichment results. High precision indicates that most of the genes identified in the GO term are relevant, while high recall means that many of the relevant genes have been detected.

Now, to better understand these results I’ll use a bar graph to plot the top 10 enrichment terms against their respective intersection sizes, as demonstrated below.

```python
sns.barplot(x='intersection_size', y='name', data=go_results.head(10))
plt.title('Top 10 Enriched GO Terms')
plt.xlabel('Intersection Size')
plt.ylabel('GO Term')
plt.show()
```

[INSERT GO4.png]

As you can see, our top GO terms include cellular components, such as collagen-containing extracellular matrix, as well as biological processes including blood vessel, vascular, and circulation system development which make sense given that we’re our data contains genes that are differentially expressed following a robust exercise stimulus. In addition to the bar graph above, there are a number of other useful visualization for GO terms including enrichment maps, gene-concept network graphs, heat maps, GO term hierarchy trees, and CIRCOS plots.
