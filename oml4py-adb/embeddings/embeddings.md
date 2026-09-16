# Lab 11: Generate and Use Text Embeddings with Pretrained ONNX Models

## Introduction

In this lab, you will learn how to load a pretrained Hugging Face model, convert it to Open Neural Network Exchange (ONNX) format, and use it within Oracle AI Database. ONNX is an open standard for representing machine learning models, allowing seamless deployment across different frameworks and environments while offering performance benefits. You will use ONNX pipeline models, import them into the database, and perform inference directly in-database.

> **Note:** This lab requires access to an on-premise Oracle AI Database.

**Estimated time:** 30 minutes

### About Bring Your Own Models

OML4Py 2.1 on Oracle Database 23.7 expands this *bring your own model* capability by supporting the automated conversion of Hugging Face text, image, and multi-modal transformers to the [Open Neural Network Exchange (ONNX)](https://onnx.ai/) format using *ONNX Pipeline Models* and the import of those models. These capabilities make it easier to leverage a broader set of text, image, and multi-modal transformer models, including embedding, text-classification, and reranking models. It also supports importing traditional ML models converted to ONNX format for use with the in-database ONNX Runtime. The ONNX Runtime eliminates the need to call separately hosted transformer models.

### Objectives

In this lab, you will learn how to:

- Install Python 3.12.6.
- Install the OML4Py 2.1.1 client.
- Load the preconfigured pretrained embedding model, `ALL_MINILM_L6`.
- Import a model into the database for inference.
- Compute text similarity using cosine similarity on embedded vectors.

### Prerequisites

- Python 3.12.6
- Oracle AI Database 26ai
- Linux x64 (OL8)

## Task 1: Install Python, OML4Py Client, and Required Packages

Prepare your machine with the necessary runtime and libraries.

1. **Install Python version** Make sure you are running the required Python version.
2. **Install third-party packages.** Typical packages include `oracledb` for database connectivity and data-science libraries such as `numpy` and `pandas`.
3. **Install Oracle Instant Client.** Required for Oracle database connectivity.
4. **Install the OML4Py 2.1.1 client.** This package provides Python interfaces for Oracle Machine Learning functions.

**Codex:** Link each installation item above to its corresponding OML4Py installation instruction. This keeps package versions, prerequisites, and platform-specific commands authoritative and current.

## Task 2: Sample Dataset Setup

This use case requires a `products` table in your Oracle AI Database containing example products and descriptions.

Create the dataset using one of the following approaches.

| product_id | name | description |
| --- | --- | --- |
| 1 | Laptop | A lightweight laptop with 16GB RAM and long battery life |
| 2 | Phone | A smartphone with great camera and AMOLED display |
| 3 | Tablet | A 10-inch tablet suitable for reading and browsing |
| 4 | Headphones | Wireless headphones with noise cancellation and deep bass |

Choose from: 

- ### Use a SQL script

    ```sql
    CREATE TABLE products (
        product_id NUMBER PRIMARY KEY,
        name VARCHAR2(100),
        description VARCHAR2(500),
        description_embedding VECTOR(384, FLOAT32)
    );

    INSERT INTO products (product_id, name, description)
    VALUES (1, 'Laptop', 'A lightweight laptop with 16GB RAM and long battery life');

    INSERT INTO products (product_id, name, description)
    VALUES (2, 'Phone', 'A smartphone with great camera and AMOLED display');

    INSERT INTO products (product_id, name, description)
    VALUES (3, 'Tablet', 'A 10-inch tablet suitable for reading and browsing');

    INSERT INTO products (product_id, name, description)
    VALUES (4, 'Headphones', 'Wireless headphones with noise cancellation and deep bass');

    COMMIT;
    ```

- ### Create using Python and OML4Py

    **Note** Update `user`, `password`, and `dsn` to match your Oracle environment. Do not put real credentials in this lab or in a notebook shared with others.

    ```python
    import pandas as pd
    import oml

    # Sample data
    data = [
        (1, 'Laptop', 'A lightweight laptop with 16GB RAM and long battery life'),
        (2, 'Phone', 'A smartphone with great camera and AMOLED display'),
        (3, 'Tablet', 'A 10-inch tablet suitable for reading and browsing'),
        (4, 'Headphones', 'Wireless headphones with noise cancellation and deep bass')
    ]
    df = pd.DataFrame(data, columns=['product_id', 'name', 'description'])

    # Connect to Oracle AI Database through OML4Py.
    oml.connect(
        user=<your_user>,
        password=<your_pass>,
        dsn=<your_host:port/service_name>
    )
    # Create the table from the pandas DataFrame in the current OML4Py session.
    oml.create(df, table="PRODUCTS")
    ```

## Task 3: Import Libraries

Import the libraries needed for database operations and machine-learning tasks.

```text
<code>
import oml
from oml.algo import onnx
</code>
```

## Task 4: Load the Preconfigured Pretrained Model

OML4Py provides access to pretrained models such as `ALL_MINILM_L6`, which are ready for embedding generation. Load a ready-to-use ONNX model to generate text embeddings.

```
<code>
model = onnx(model_name="ALL_MINILM_L6")
</code>
```

## Task 5: Load the Model to Your Database

The model can be exported in either of the following ways:

- Use `export2db` to export the model directly from Python to the connected database.
- Use `export2file` to create a local ONNX file and then import that file into the database with `DBMS_VECTOR.LOAD_ONNX_MODEL`.

**Codex:** Add the exact `export2db` or `export2file` call from [Step 10 of the OML4Py pretrained-model conversion instructions](https://docs.oracle.com/en/database/oracle/machine-learning/oml4py/2-23ai/mlpug/convert-pretrained-models-onnx-model-end-end-instructions.html), using the model name selected for this lab.

## Task 6: Generate and Store Text Embeddings

Use the pretrained model to convert each product description into a numerical vector, known as an embedding. These embeddings capture semantic meaning and can be compared for similarity-search and recommendation use cases.

```python
<code>
# Run after ALL_MINILM_L6 has been loaded into this database.
UPDATE products p
SET description_embedding = vector_embedding(
    ALL_MINILM_L6 USING p.description AS DATA
);

COMMIT;
</code>
```

**Codex:** This step is required. The original lab generated an embedding in a query result but never stored it, while the later similarity query reads `description_embedding`.

## Task 7: Convert a Single Text to an Embedding

Transform arbitrary text, such as a user query, into an embedding. This allows the query's meaning to be compared with existing product embeddings.

```python
new_text = "music noise reduction headphones"
# The actual embedding happens in the SQL query in Task 7.
```

## Task 8: Compute Similarity with Existing Embeddings Using Cosine Similarity

After converting the product descriptions and a new text input into embeddings, compute vector distance to find product descriptions that are closest in meaning to the user input. This enables search, recommendation, and matching features.

```python
query = """
WITH q AS (
    SELECT vector_embedding(ALL_MINILM_L6 USING :txt AS data) AS qvec
    FROM dual
)
SELECT p.product_id,
       p.name,
       p.description,
       vector_distance(p.description_embedding, q.qvec, COSINE) AS similarity
FROM products p, q
ORDER BY similarity ASC
FETCH FIRST 3 ROWS ONLY
"""

-- Codex: Supply `new_text` as the bind variable `:txt` in your SQL client or OML4Py execution method.
```

**Codex:** Expected result: the query returns up to three products ordered from the smallest cosine distance (most similar) to the largest distance.

## Optional Task: Load Nonconfigured Models

1. Set the configuration.
2. Load the model.

**Codex:** Decide whether this optional task is in scope. If it is, add one complete, tested example that configures a model with an Oracle template and then loads it; otherwise, remove this incomplete section.

## Learn More

- [Get Started with Oracle Machine Learning for Python](https://docs.oracle.com/en/database/oracle/machine-learning/oml4py/1/mlpug/get-started-with-oml4py.html#GUID-B45A76E6-CE48-4E49-B803-D25CA44B09ED)
- [Oracle Machine Learning Notebooks](https://docs.oracle.com/en/database/oracle/machine-learning/oml-notebooks/)

## Acknowledgements

- **Author** - Dhanish Kumar, Senior Member of Technical Staff
- **Contributors** -  Mark Hornick, Senior Director, Data Science and Machine Learning; Sherry LaMonica, Principal Member of Tech Staff, Advanced Analytics, Machine Learning
- **Last Updated By/Date** - Dhanish Kumar, August 2026

