# Lab 10: Create, Manage, and Analyze Vector Data

## Introduction

This lab walks you through the steps to use the OML4Py vector data type, `oml.vector`. The class `oml.vector` represents a single column of vector data in an Oracle AI Database table or view, and can be part of OML4Py DataFrame proxy objects. DataFrame proxy objects enable manipulating database data without loading the table data into Python memory.

Estimated Time: 15 minutes

### About vector type support

Vectors are compact semantic representations of unstructured data like text and images, and play a key role in Oracle Database 26ai AI Vector Search, for which the vector datatype was introduced. OML4Py 2.1 includes `oml.vector`, which can be part of OML4Py DataFrame proxy objects, whether reading from or writing to the database or for manipulation using OML4Py functions. Vector columns can also be used with compatible machine learning algorithms for model training and prediction. This capability facilitates using vectors as predictors and producing vectors, which can then be used for similarity searches with AI Vector Search.

### Objectives

In this lab, you will:

* Use `oml.create` to efficiently store and manage vector data in the Oracle Database with the VECTOR datatype.
* Utilize OML4Py methods such as `typecode()` and `dimension()` to inspect and validate vector column metadata in-database.
* Prepare and split data for modeling using `oml.sync()` and `oml.split()`.
* Build and train a K-Means clustering model with `oml.km()` directly on data stored in Oracle.
* Evaluate model results and use the model’s `predict()` method for scoring new data.

### Prerequisites

* Oracle Database 26ai with OML4Py 2.1 installed.
* An active OML4Py connection to an Oracle AI Database. Connect before running the lab code.
* Python packages: `pandas`, `numpy`, and `scikit-learn`.

<!-- ## Tasks -->

## Task 1: Create and explore vector data in the database

To store and query vector datatypes using OML4Py, create a simple pandas DataFrame that includes both a vector column and a label column. Use the `oml.create` function to persist this DataFrame into the Oracle Database, specifying the VECTOR type for the appropriate column. Once the data is stored, use OML4Py’s built-in methods to inspect the in-database vector column, such as retrieving its data type and dimensionality, ensuring that the data has been correctly represented and enabling further analysis within the database environment.

- Create a sample dataset with vector datatype columns using `oml.create`.
- Demonstrate operations like `typecode` and `dimension()` on vector data.

    ```text
    <copy>
    import oml
    import pandas as pd
    import numpy as np

    # Create a pandas DataFrame with a VECTOR column and a label column
    sample_df = pd.DataFrame({
        'VECTOR': [
            np.array([1.0, 2.0, 3.0, 4.0]),
            np.array([5.0, 6.0, 7.0, 8.0]),
            np.array([2.5, 4.5, 6.5, 8.5])
        ],
        'LABEL': ['A', 'B', 'C']
    })

    # Create and push to Oracle Database as a vector column
    sample_tbl = oml.create(
        sample_df,
        table='SAMPLE_VECTOR_TBL',
        dbtypes=['VECTOR(4, float32)', 'VARCHAR2(10)']
    )

    # Select the VECTOR column as a proxy object
    vector_proxy = sample_tbl[['VECTOR']]

    # Explore typecode: Shows the internal type
    print("Typecode:", vector_proxy.typecode())

    # Explore dimension: Shows the dimensionality of the vector
    print("Dimension:", vector_proxy.dimension())
    </copy>
    ```

    Expected output:
    <!-- # (1007 is the typecode for VECTOR in OML4Py) -->

    ```text
    Typecode: [1007]  
    Dimension: [4]
    ```

## Task 2: Import libraries and load the Iris dataset

Import the required Python libraries and load the Iris dataset from `scikit-learn`. This dataset contains flower measurements that will be used for vector-based machine learning examples.
```text
<copy>
import pandas as pd
from sklearn.datasets import load_iris

# Load the Iris dataset
iris = load_iris()
</copy>
```

## Task 3: Create a vector column from the dataset

To enable efficient vector-based operations and simplify downstream machine learning tasks, it is useful to consolidate multiple numeric features into a single vector column. Creating separate DataFrames for the feature values and the labels—one for the four numeric feature columns and another for the species name of each sample. Then, concatenate these DataFrames to form `iris_df` and add a new column named `VECTOR`, which stores a list of feature values as a single vector for each row.

```text
<copy>
# Create feature DataFrame
x = pd.DataFrame(iris.data, columns=['SEPAL_LENGTH', 'SEPAL_WIDTH', 'PETAL_LENGTH', 'PETAL_WIDTH'])

# Create target DataFrame
y = pd.DataFrame(list(map(lambda x: {0: 'setosa', 1: 'versicolor', 2: 'virginica'}[x], iris.target)), columns=['SPECIES'])

# Combine features and target columns
iris_df = pd.concat([x, y], axis=1)

# Create a VECTOR column from the numeric feature columns
iris_df['VECTOR'] = iris_df[['SEPAL_LENGTH', 'SEPAL_WIDTH', 'PETAL_LENGTH', 'PETAL_WIDTH']].values.tolist()

vector_df = iris_df[['VECTOR', 'SPECIES']]
print(vector_df.head())
</copy>
```

Expected output:

```text
                 VECTOR SPECIES
0  [5.1, 3.5, 1.4, 0.2]  setosa
1  [4.9, 3.0, 1.4, 0.2]  setosa
2  [4.7, 3.2, 1.3, 0.2]  setosa
3  [4.6, 3.1, 1.5, 0.2]  setosa
4  [5.0, 3.6, 1.4, 0.2]  setosa
```

## Task 4: Store prepared data and split for training/test

To prepare the data for modeling, first select the `VECTOR` and `SPECIES` columns from `iris_df` to create a new DataFrame called `vector_df`. Store this DataFrame in the Oracle Database using `oml.create`, specifying explicit data types for each column and naming the table, `vector_df`. Once the data is stored, retrieve a proxy object for the table and split the data into training and test sets, enabling you to build and evaluate machine learning models directly in the database.

```text
<copy>
vector_df = iris_df[['VECTOR', 'SPECIES']]

# Create the table and proxy object with explicit dbtypes
vector_tbl = oml.create(
    vector_df,
    table='vector_df',
    dbtypes=['VECTOR(4, float32)', 'VARCHAR2(20)']
)
</copy>
```

## Task 5: Retrieve and split the data for training and testing

Retrieve a proxy object for the database table using `oml.sync()`. Then split the data into training and test sets using `.split()`, which creates a 60/40 train-test split by default. The split operation is performed directly in the Oracle AI Database, and the resulting training and test datasets are returned as proxy objects.

```text
<copy>
# Retrieve a proxy object for the database table
vector_proxy = oml.sync(table='vector_df')

# Split the data into training and test sets (60/40 by default)
train_dat, test_dat = vector_proxy.split()

print(train_dat.head())
</copy>
```

Expected output:

```text
                                              VECTOR     SPECIES
0  [5.599999904632568, 2.700000047683716, 4.19999...]  versicolor
1  [5.699999809265137, 3.0, 4.199999809265137, 1....]  versicolor
2  [5.699999809265137, 2.799999952316284, 4.09999...]  versicolor
3  [6.300000190734863, 3.299999952316284, 6.0, 2.5]   virginica
4  [5.800000190734863, 2.700000047683716, 5.09999...]   virginica

```

## Task 6: Train a K-Means model on vector data

To leverage in-database modeling on vector data, begin by defining the settings for the K-Means algorithm, such as specifying the number of maximum iterations. Next, create and train a K-Means clustering model with three clusters using the training dataset. Once the model has been trained, print its details to review information about the clustering process and resulting cluster assignments.

After training, print the model object to review cluster assignments, algorithm settings, and summary statistics. These details help evaluate the clustering results and overall model behavior.

```text
<copy>
# Define K-Means settings
setting = {'kmns_iterations': 20}

# Train the K-Means model with 3 clusters
km_mod = oml.km(n_clusters=3, **setting).fit(train_dat)

# Review model details and clustering summary
print(km_mod)
</copy>
```

Expected output:

```text
Algorithm Name: K-Means

Mining Function: CLUSTERING

Settings: 
                    setting name            setting value
0                      ALGO_NAME              ALGO_KMEANS
1              CLUS_NUM_CLUSTERS                        3
2            KMNS_CONV_TOLERANCE                     .001
3                   KMNS_DETAILS   KMNS_DETAILS_HIERARCHY
4                  KMNS_DISTANCE           KMNS_EUCLIDEAN
5                KMNS_ITERATIONS                       20
6      KMNS_MIN_PCT_ATTR_SUPPORT                       .1
7                  KMNS_NUM_BINS                       11
8               KMNS_RANDOM_SEED                        0
9           KMNS_SPLIT_CRITERION            KMNS_VARIANCE
10                KMNS_WINSORIZE   KMNS_WINSORIZE_DISABLE
11                  ODMS_DETAILS              ODMS_ENABLE
12  ODMS_MISSING_VALUE_TREATMENT  ODMS_MISSING_VALUE_AUTO
13                 ODMS_SAMPLING    ODMS_SAMPLING_DISABLE
14                     PREP_AUTO                       ON

Computed Settings: 
              setting name setting value
0  ODMS_EXPLOSION_MIN_SUPP             1

Global Statistics: 
   attribute name attribute value
0       CONVERGED             YES
1  NUM_ITERATIONS               3
2        NUM_ROWS             108

Attributes: 
SPECIES
VECTOR

Partition: NO

Clusters: 

   CLUSTER_ID  ROW_CNT  PARENT_CLUSTER_ID  TREE_LEVEL  DISPERSION
0           1      108                NaN           1    0.591106
1           2       35                1.0           2    0.331967
2           3       73                1.0           2    0.715351
3           4       36                3.0           3    0.772299
4           5       37                3.0           3    0.659942

Taxonomy: 

   PARENT_CLUSTER_ID  CHILD_CLUSTER_ID
0                  1               2.0
1                  1               3.0
2                  2               NaN
3                  3               4.0
4                  3               5.0
5                  4               NaN
6                  5               NaN

Leaf Cluster Counts: 

   CLUSTER_ID  CNT
0           2   35
1           4   36
2           5   37

```

## Task 7: Review model performance

To review model performance, display the details of the trained K-Means model by printing `km_mod`. This output includes cluster assignments, model parameters, and summary statistics, providing valuable insights into the clustering results and overall effectiveness of the model.

```text
<copy>
print(km_mod.cluster)  # automatically includes cluster details and summary stats
</copy>
```

Expected output:

```text
   CLUSTER_ID  ROW_CNT  PARENT_CLUSTER_ID  TREE_LEVEL  DISPERSION
0           1      108                NaN           1    0.591106
1           2       35                1.0           2    0.331967
2           3       73                1.0           2    0.715351
3           4       36                3.0           3    0.772299
4           5       37                3.0           3    0.659942
```

## Task 8: Predict cluster assignments on test data

To score the model on new or unseen data, use the trained K-Means model to predict cluster assignments for the test dataset. Then, display the first few predictions using `head()` to review how the model has categorized these new samples and assess the effectiveness of its clustering on data it has not seen before.

```text
<copy>
predictions = km_mod.predict(test_dat, supplemental_cols=test_dat[:, ['VECTOR', 'SPECIES']])
print(predictions.head())
</copy>
```

Expected output:

```text
                                               VECTOR     SPECIES  CLUSTER_ID
0    [5.0, 2.299999952316284, 3.299999952316284, 1.0]  versicolor           5
1   [5.699999809265137, 2.9000000953674316, 4.1999..]  versicolor           5
2   [6.199999809265137, 2.9000000953674316, 4.3000..]  versicolor           5
3    [5.099999904632568, 2.5, 3.0, 1.100000023841858]  versicolor           5
4   [7.599999904632568, 3.0, 6.599999904632568, 2..]   virginica            4

```

## Learn More

* [Get Started with Oracle Machine Learning for Python](https://docs.oracle.com/en/database/oracle/machine-learning/oml4py/1/mlpug/get-started-with-oml4py.html#GUID-B45A76E6-CE48-4E49-B803-D25CA44B09ED)
* [Oracle Machine Learning Notebooks](https://docs.oracle.com/en/database/oracle/machine-learning/oml-notebooks/)

## Acknowledgements

* **Author** - Dhanish Kumar, Senior Member of Technical Staff
* **Contributors** -  Mark Hornick, Senior Director, Data Science and Machine Learning; Sherry LaMonica, Principal Member of Tech Staff, Advanced Analytics, Machine Learning
* **Last Updated By/Date** - Dhanish Kumar, August 2026
