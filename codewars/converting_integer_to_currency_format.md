# Converting integer to currency format

## My Solution
```js
function toCurrency(price){
  var priceArr = (""+price).split('');
  var formattedPrice = [];
  var cnt = 0;
  while (priceArr.length) {
    formattedPrice.push(priceArr.pop());
    
    if ( cnt++ == 2 && priceArr.length ) {
      formattedPrice.push(',')
      cnt = 0;
    }
  }
  
  priceArr = [];
  while(formattedPrice.length) {
    priceArr.push(formattedPrice.pop());
  }
  
  return priceArr.join('');
}
```

하지만...

정규표현식이 갑!

```js
function toCurrency(price){
  return price.toString().replace(/(\d)(?=(\d{3})+$)/g, '$1,');
}
```