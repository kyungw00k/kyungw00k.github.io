# Credit Card Mask

마지막 4자리를 제외한 나머지 부분만 #로 Masking 처리 하는 문제. the last four characters into '#'.

## Examples

```
maskify("4556364607935616") == "############5616"
maskify(     "64607935616") ==      "#######5616"
maskify(               "1") ==                "1"
maskify(                "") ==                 ""

// "What was the name of your first pet?"
maskify("Skippy")                                   == "##ippy"
maskify("Nananananananananananananananana Batman!") == "####################################man!"
```

## 제출한 Solution

```js
// return masked string
function maskify(cc) {
  return ( cc.length > 4 )
    ? [ cc.substr(0,cc.length - 4).replace(/[A-Za-z\d]/g,'#'), cc.substr(cc.length - 4 ,4)].join('')
    : cc;
}

```

## 다른 답을 보니...
역시 정규표현식은 진리인듯. #에라이