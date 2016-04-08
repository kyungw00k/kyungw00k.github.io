# 템플릿에서 날짜 다루기


## Intro
Airflow Job에서 Hive 쿼리를 돌리는데 기준 날짜에 `YYYY-MM-DD` 포멧인 `ds`를 사용하고 있었다.
포멧을 바꿀때는 `ds`를 쓰지 않고 `execution_date.strftime("%Y%m%d")`로 사용했는데, `ds`와 `execution_date`의 혼용이 맞나 싶어서 한쪽으로 몰 수 있는 방법이 없나 싶어서 찾아봤다.

## `execution_date`로 몰아서 사용하기
사실 방법이 가장 간단하고 빠르게 적용할 수 있고, 각 날짜의 패턴을 명시적으로 알 수 있다는 장점이 있지만, 매번 포멧을 써줘야하는 단점도 있다.

## Airflow에서 제공하는 `macros` 사용하기
Airflow쪽 이슈나 [코드](https://github.com/airbnb/airflow/blob/58029df26efb58e75e82fe032a60532a25dc93d8/airflow/macros/__init__.py)를 찾아보니 `ds`를 조작하는 함수가 있었습니다.

### `macros.ds_add(ds, days)`
```py
def ds_add(ds, days):
    """
    Add or subtract days from a YYYY-MM-DD
    :param ds: anchor date in ``YYYY-MM-DD`` format to add to
    :type ds: str
    :param days: number of days to add to the ds, you can use negative values
    :type days: int
    >>> ds_add('2015-01-01', 5)
    '2015-01-06'
    >>> ds_add('2015-01-06', -5)
    '2015-01-01'
    """

    ds = datetime.strptime(ds, '%Y-%m-%d')
    if days:
        ds = ds + timedelta(days)
    return ds.isoformat()[:10]
```

다음날은 `macros.ds_add(ds, 1)`로, 하루 전은 `macros.ds_add(ds, -1)`로 사용하면 되겠네요.

참고로, `execution_date`를 사용한다면 `macros.timedelta`의 조합으로 하시면 될 것 같습니다.

따라서, 아래 두 표현은 같은 값이 되겠네요.

```py
# 다음 날
macros.ds_add(ds, 1)
execution_date+macros.timedelta(1).strftime("%Y-%m-%d")
```

### `macros.ds_format(ds, input_format, output_format)`
```py
def ds_format(ds, input_format, output_format):
    """
    Takes an input string and outputs another string
    as specified in the output format
    :param ds: input string which contains a date
    :type ds: str
    :param input_format: input string format. E.g. %Y-%m-%d
    :type input_format: str
    :param output_format: output string format  E.g. %Y-%m-%d
    :type output_format: str
    >>> ds_format('2015-01-01', "%Y-%m-%d", "%m-%d-%y")
    '01-01-15'
    >>> ds_format('1/5/2015', "%m/%d/%Y",  "%Y-%m-%d")
    '2015-01-05'
    """
    return datetime.strptime(ds, input_format).strftime(output_format)
```

간단한 예로, `ds`가 `YYYY-MM-DD` 포멧인데 이를 `YYYYMMDD`로 바꾸고자 할 때 사용하면 됩니다.

`macros.ds_format(ds, "%Y-%m-%d", "%Y%m%d")`