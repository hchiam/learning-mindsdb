# Learning MindsDB

Just one of the things I'm learning. https://github.com/hchiam/learning

Write SQL-like code to create AI tables to embed ML models directly into your DataBase: train, then `SELECT` to predict. Automatic ML Ops.

Fireship.io intro: https://www.youtube.com/watch?v=jb2AvF8XzII&t=469s

Official GitHub repo: https://github.com/mindsdb/mindsdb

Official website: https://mindsdb.com

Quickstart: https://docs.mindsdb.com/quickstart

Example tutorial: https://docs.mindsdb.com/nlp/nlp-mindsdb-openai

## Generally:

```sql
CREATE MODEL project_name.predictor_name -- AI table
PREDICT target_column -- column to store predictions
USING
  engine = 'openai',
  prompt_template = 'prompt/task
                    {{input_column}}', -- prompt template with placeholders
  model_name = 'model_name', -- (optional) default is text-davinci-002
  api_key = 'YOUR_OPENAI_API_KEY'; -- (optional) default is `OPENAI_API_KEY`  env var
```

## Specific example, with SQL:

```sql
CREATE MODEL mindsdb.sentiment_classifier
PREDICT sentiment
USING
  engine = 'openai',
  prompt_template = 'predict the sentiment of the text:{{review}} exactly as either positive or negative or neutral';

-- verify that the AI table sentiment_classifier was properly initialized:
SELECT *
FROM mindsdb.models
WHERE name = 'sentiment_classifier';
```

Then:

```sql
-- CREATE DATABASE example_db
-- WITH ENGINE = "postgres",
-- PARAMETERS = {
--     "user": "demo_user",
--     "password": "demo_password",
--     "host": "3.220.66.106",
--     "port": "5432",
--     "database": "demo"
--     };

-- predict sentiment and show the original review too:
SELECT output.sentiment, input.review
FROM example_db.demo_data.amazon_reviews AS input
JOIN mindsdb.sentiment_classifier AS output
LIMIT 3;
```

## Tutorial - predict home rental prices:

Instead of [using pip](https://docs.mindsdb.com/setup/self-hosted/pip/source) or [running a local docker image](https://docs.mindsdb.com/setup/self-hosted/docker), you can [create an account](https://cloud.mindsdb.com/login) to run an instance in the cloud.

Connect demo database `example_db` to `mindsdb`:

```sql
CREATE DATABASE example_db
WITH ENGINE = "postgres",
PARAMETERS = {
    "user": "demo_user",
    "password": "demo_password",
    "host": "3.220.66.106",
    "port": "5432",
    "database": "demo"
    };
```

Preview the data that's in `example_db`: (click "Data insights" for data visualization)

```sql
SELECT *
FROM example_db.demo_data.home_rentals
LIMIT 10;
```

Create and train model `home_rentals_model` inside of `mindsdb`, using `example_db` data to predict `rental_price`:

```sql
CREATE MODEL
  mindsdb.home_rentals_model
FROM example_db
  (SELECT * FROM demo_data.home_rentals)
PREDICT rental_price;
```

- [AutoML](https://github.com/hchiam/learning-automl) is used by default
- To see how the model was built: `DESCRIBE MODEL mindsdb.predictor_name.model;` https://docs.mindsdb.com/sql/api/describe?h=describe
- To have control over how the model was built: `USING ... = ..., ... = ..., ...` https://docs.mindsdb.com/sql/create/predictor?h=usin#using-statement

Check model training status:

```sql
SELECT *
FROM mindsdb.models
WHERE name='home_rentals_model';
```

Predict `rental_price` (with "explanations" like confidence from `rental_price_explain`) from `home_rentals_model` from `mindsdb`, give some `WHERE` inputs:

```sql
SELECT rental_price,
       rental_price_explain
FROM mindsdb.home_rentals_model
WHERE sqft = 823
AND location='good'
AND neighborhood='downtown'
AND days_on_market=10;
```

Bulk predict:

```sql
SELECT t.rental_price as real_price,
m.rental_price as predicted_price,
t.number_of_rooms,  t.number_of_bathrooms, t.sqft, t.location, t.days_on_market
FROM example_db.demo_data.home_rentals as t
JOIN mindsdb.home_rentals_model as m limit 100;
```
