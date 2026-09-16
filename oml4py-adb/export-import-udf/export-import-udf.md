# Lab 12: Export and Import user-defined Python Scripts

This lab shows you how to manage, share, and migrate user-defined Python functions (UDFs) using the OML4Py script repository. You will learn to export UDFs from a database, import them when needed, and verify the process. You can also invoke UDFs using OML4Py embedded Python execution from Python, SQL, and REST APIs.

> **Note**: This lab requires access to an on-premise Oracle AI Database.

**Estimated time:** 15 minutes

## About Export and Import User-defined Python Scripts

The Python Script Repository enables you to store user-defined functions (UDFs) directly in the Oracle Database, making them available for immediate use in your Python sessions. With OML4Py, you can invoke these UDFs not only from Python, but also through SQL and REST APIs, supporting a wide range of applications.

To facilitate sharing and migrating scripts from one database to another, user-defined Python functions, also called scripts, can be exported or imported among Python script repositories across database instances and schemas. This simplifies solution deployment—for example, deploying from development to test and then production systems.

### Objectives

In this lab, you will:

- List available user-defined Python functions (UDFs) in your OML4Py script repository.
- Export and import UDFs between databases/schemas using Python and JSON script formats.
- Drop (remove) your private UDFs if needed.
- Verify the existence of exported/imported UDF files.

### Prerequisites

To complete this lab, ensure you have access to:

- OML4Py 2.1
- Oracle AI Database 26ai (or later)
- Required permissions to use the script repository features
- Required database privilege: The database account used by the OML4Py session must have the `PYQADMIN` role before running `oml.script.create(...)`. A DBA or other authorized administrator should run the following statement, substituting the correct database user if it is not `oml_user`:

    ```sql
    GRANT PYQADMIN TO oml_user;
    ```
    Reconnect the OML4Py session after the role is granted, then rerun the relevant `oml.script.create(...)` call.

## Task 0: Import Required Libraries

Import the required OML and Python libraries. The oml library provides access to OML4Py features, while os is used later in the lab to verify that exported script files were created successfully.

```
<copy>
import oml
import os
</copy>
```

## Task 1: Create and Upload a User-defined Function (UDF)

Create and upload a Python user-defined function (UDF) to the Oracle AI Database using the OML4Py script repository. 

> **Note**: Ensure that the database user used for this connection has the `PYQADMIN` role, which is required to create, export, import, and manage scripts in the repository.

The **`is_global`** argument determines the scope of the function:

- **`is_global=False`** (default): The function is private and available only to the current session user.
- **`is_global=True`**: The function is global and available to all users with read and execute privileges.

Upload the functions `build_lm1` and `build_lm2` to the database as shown below, creating `build_lm1` as a user-level script and `build_lm2` as a global-level script.

### **User-level script**: `build_lm1`

```python
<copy>
# Define a function.
build_lm1_user = '''def build_lm1_user(dat):
    import pandas as pd
    from sklearn import linear_model
    regr = linear_model.LinearRegression()
    dat = pd.get_dummies(dat, drop_first=True)
    X = dat[["Sepal_Width", "Petal_Length", "Petal_Width", "Species_versicolor", "Species_virginica"]]
    y = dat[["Sepal_Length"]]
    regr.fit(X, y)
    return regr'''

# Create a private user-defined Python function
oml.script.create(
    name="build_lm1_user",
    func=build_lm1_user,
    description="Train a linear regression model on Iris features (returns coefficients/intercept)",
    overwrite=True
)
</copy>
```

The script creates the private `build_lm1_user` script in your repository.

### **Global-level script**: `build_lm2`

```python
<copy>
# Define another function
build_lm2_global = '''def build_lm2_global(dat):
    from sklearn import linear_model
    regr = linear_model.LinearRegression()
    X = dat[["Petal_Width"]]
    y = dat[["Petal_Length"]]
    regr.fit(X, y)
    return regr'''

# Save the function as a global script to the script repository, overwriting any existing function with the same name.
oml.script.create(
    name="build_lm2_global",
    func=build_lm2_global,
    is_global=True,
    description="Train a linear regression model on Iris features (returns coefficients/intercept)",
    overwrite=True
)
</copy>
```

The script completes creates the global `build_lm2_global` script. Users need the appropriate read and execute privileges to use it.

## Task 2: List UDFs in the Script Repository

Before exporting or importing, it’s important to know what scripts you have access to.

```python
<copy>
# List all UDFs containing "LM" in their name, for all script types (user and global)
oml.script.dir(name="lm", regex_match=True, sctype="all")[['owner', 'name', 'script', 'description']]
</copy>
```

Expected output:

```
<copy>
      owner  ...                                        description
0  OML_USER  ...  Train a linear regression model on Iris featur...
1    PYQSYS  ...  Train a linear regression model on Iris featur...

[2 rows x 4 columns]
</copy>
```

A table is returned with columns `owner`, `name`, `script`, and `description`. It includes `build_lm1_user` (owned by the current user) and `build_lm2_global` if you have access to the global script. The script body is shown in the `script` column.

## Task 3: Export UDFs from the Script Repository

You can export user-defined functions (UDFs) from the OML4Py script repository to a local file for backup, sharing, or migration. Use the `oml.export_script` function and choose one of the following export formats:

- **JSON (`.json`):** Exports scripts as a list of dictionaries containing the script’s name, function, and description.
- **Python (`.py`):** Exports scripts in executable Python format, each with docstrings that include the script name and description.

```python
<copy>
import os

# Export user-owned scripts to a JSON file
oml.export_script(file_name="build_lm1_user.json", sctype="user")

# Export global scripts to a Python file
oml.export_script(file_name="build_lm2_global.py", sctype="global")

# Verify that export files exist in the current directory
{"build_lm1_user.json", "build_lm2_global.py"}.issubset(os.listdir(path="./"))
</copy>
```
Expected output:
```
True
```


Both export calls completes and the final expression returns `True`, confirming that `build_lm1_user.json` and `build_lm2_global.py` exist in the current working directory.

## Task 4: Import UDFs into the Script Repository

Import previously exported UDFs to make them available in a new environment or schema.

```python
<copy>
# Import private UDFs from JSON file
oml.import_script(file_name="build_lm1_user.json")

# List your currently available UDFs
oml.script.dir()[['name', 'script', 'description']]

# Import global UDFs from Python file and make them available to all users
oml.import_script(file_name="build_lm2_global.py", is_global=True, overwrite=True)

# Confirm the imported UDFs are present
oml.script.dir(name="lm", regex_match=True, sctype="all")[['owner', 'name', 'script', 'description']]
</copy>
```
Expected output:
```
      owner  ...                                        description
0  OML_USER  ...  Train a linear regression model on Iris featur...
1    PYQSYS  ...  Train a linear regression model on Iris featur...

[2 rows x 4 columns]
```

The imports completes. The first listing includes `build_lm1_user`and the final listing includes both `build_lm1_user` and `build_lm2_global`, with their respective owner and description details.

## Task 5: Drop (Remove) a Private UDF

If you no longer need a private UDF, you can remove it.

```text
<copy>
# Drop a specific private UDF by its script name
oml.script.drop(name="build_lm1_user")
</copy>
```

After the call completes, rerunning the Task 2 listing no longer returns `build_lm1_user`; the global `build_lm2_global` script remains unaffected.

## Learn More

- [Get Started with Oracle Machine Learning for Python](https://docs.oracle.com/en/database/oracle/machine-learning/oml4py/1/mlpug/get-started-with-oml4py.html#GUID-B45A76E6-CE48-4E49-B803-D25CA44B09ED)
- [Oracle Machine Learning Notebooks](https://docs.oracle.com/en/database/oracle/machine-learning/oml-notebooks/)

## Acknowledgements

- **Author** - Dhanish Kumar, Senior Member of Technical Staff
- **Contributors** -  Mark Hornick, Senior Director, Data Science and Machine Learning; Sherry LaMonica, Principal Member of Tech Staff, Advanced Analytics, Machine Learning
- **Last Updated By/Date** - Dhanish Kumar, August 2026
