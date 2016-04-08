# 템플릿에서 날짜 다루기


## Intro
Airflow Job에서 Hive 쿼리를 돌리는데 기준 날짜에 `YYYY-MM-DD` 포멧인 `{{ ds }}`를 사용하고 있었다.
포멧을 바꿀때는 `ds`를 쓰지 않고 `execution_date.strftime("%Y%m%d")`로 사용했는데, `ds`와 `execution_date`의 혼용이 맞나 싶어서 한쪽으로 몰 수 있는 방법이 없나 싶어서 찾아봤다.

## `execution_date`로 몰아서 사용하기
사실 방법이 가장 간단하고 빠르게 적용할 수 있고, 각 날짜의 패턴을 명시적으로 알 수 있다는 장점이 있지만, 매번 포멧을 써줘야하는 단점도 있다.

## Airflow에서 제공하는 `macros` 사용하기
Airflow쪽 이슈나 [코드](https://github.com/airbnb/airflow/blob/58029df26efb58e75e82fe032a60532a25dc93d8/airflow/macros/__init__.py)를 찾아보니, 