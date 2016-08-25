---
layout: post
title: Setup Airflow on Mac
date: 2016-05-04 16:09:38.000000000 +09:00
type: post
categories:
- Airflow
- Mac
---
## Setup on Mac
`brew` is mandatory!

```sh
brew install python python3
pip install airflow

mkdir ~/airflow

# airflow needs a home, ~/airflow is the default,
# but you can lay foundation somewhere else if you prefer
# (optional)
export AIRFLOW_HOME=~/airflow

# install from pypi using pip
pip install airflow

# initialize the database
cd ~/airflow && airflow initdb

# start the web server, default port is 8080
cd ~/airflow && airflow webserver -p 8080
```
