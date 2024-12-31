---
layout: post
title:  "GPG signature of commits in GitHub"
date:   2024-12-31 10:00:00 +0100
categories: ipython sql vscode
tags:   iPython SQL vsCode
---
Yesterday, during working hours, I was studying for a data engineering certifation, which I need to know in my new role.
To practice anything in this course, I wanted to practice - however not on the company server. It is possible to set up a data engineering platform on cloud services such as Azure, however I do not have the money to invest in something like that, nor do I want to use the bulky Jupyter notebooks.
So, I needed a solution to this problem. I needed to set a platform like databricks on my own computer. I wanted to use python in notebooks, but also be able to use SQL in the same notebook, without using Jupyter, Datalore, Collab, or other cloud or bulky software. This post describes that attempt on a macbook.

<!--more-->
## Setting up a project in VS Code
First, lets create the project directory:

```bash
mkdir notebooks
cd notebooks
```

Then I installed a new project an package manager for Python:
```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

...and initialise the project: 
```bash
uv python list
uv init --name notebooks --vcs git --author-from git --python cpython-3.13.1-macos-x86_64-none
```

In this project I will be using Python and iPython in VS Code, so
```bash
uv pip install ipython ipykernel
source ./.venv/bin/activate
```
Now we can open this project in VS Code and run de code.

## SQL in the notebooks
Next, we wanna use a database as well, locally. but without creating a database connection manually. For this we will use a plugin for iPython. Let's install even more dependencies.
```bash
brew install postgresql libpq
brew services start postgresql@14
uv pip install ipython-sql SQLAlchemy psycopg2
```

Now this almost works. However, if you try to create a connection, as described below, you will get an error saying that a file is missing. So, lets find and create a softlink to that file.
```bash
which postgres
# /usr/local/bin/postgres

ls -Alh /usr/local/bin/postgres
# /usr/local/bin/postgres -> ../Cellar/postgresql@14/14.15/bin/postgres

cd /usr/local/Cellar/libpq/17.2/

find . -type f -iname "*dylib"
# ./lib/libpq.5.dylib # this is that file
# ./lib/libecpg.6.dylib
# ./lib/libpgtypes.3.dylib
# ./lib/libecpg_compat.3.dylib

ln -s /usr/local/Cellar/libpq/17.2/lib/libpq.5.dylib /usr/local/lib/libpq.5.dylib
```

## Setting up a database
Lets set up a database
```bash
psql postgres
```

Lets create a user
```sql
CREATE USER etl_user;
ALTER USER etl_user WITH PASSWORD '5ome_h4rd_P4s5w0r7';
```

We can list the databases/schemas using `\l`. but we can also just create one like so:
```sql
CREATE DATABASE etl_test;
\c etl_test
CREATE TABLE cars (
    id BIGSERIAL PRIMARY KEY,
    model VARCHAR(255),
    year INT
);
```

And fill that database manually:
```sql
INSERT INTO cars (model, year) VALUES ('test_2', 2024);
INSERT INTO cars (model, year) VALUES ('test', 2008);
INSERT INTO cars (model, year) VALUES ('test_2', 2020);
INSERT INTO cars (model, year) VALUES ('test_3', 2024);
INSERT INTO cars (model, year) VALUES ('test_2', 2021);
INSERT INTO cars (model, year) VALUES ('test', 2019);
```

## Working in the notebook in VS Code
We first need to load the iPython plugin, and set its table style:
```ipynb
%load_ext sql
%config SqlMagic.style = '_DEPRECATED_DEFAULT'
```

I will set the variables in an .env file:
```text
.env
PSQL_USER=vannijnatten
PSQL_PASSWORD=postgresql
PSQL_SCHEMA=vannijnatten
```

Then create a connection string for the SQL plugin:
```python
import os

user = os.getenv('PSQL_USER')
password = os.getenv('PSQL_PASSWORD')
schema = os.getenv('PSQL_SCHEMA')
connection_string = f"postgresql://{user}:{password}@localhost/{schema}"
print(f"{connection_string=}")
%sql $connection_string
```

And test the connection
```sql
%%sql
SELECT id, model, year FROM cars;
```

...or...
```sql
%%sql
SELECT
    id,
    model,
    year,
    rank() OVER w,
    sum(year) OVER w
FROM cars
WINDOW w AS (PARTITION BY model ORDER BY year)
ORDER BY model, rank;
```

For me, this works and is very lightweight, in my favorite IDE (and in an environment in Python, so a clean setup).
