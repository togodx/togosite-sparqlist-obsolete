# p value (Fisher's exact test 2x2)

## Parameters

* `data`
  * default: 28,17,12,25

## `pval`
```javascript
({data})=>{
  let tmp = data.replace(/\s/, "").split(/,/);
  let a = Number(tmp[0]);
  let b = Number(tmp[1]);
  let c = Number(tmp[2]);
  let d = Number(tmp[3]);
  
  // 不正数値検出
  const maxLimit = 300000; // ensembl_transcript: ~253,000
  if (a < 0 || b < 0 || c < 0 || d < 0) return {error: "limit >= 0"};
  if (a > maxLimit || b > maxLimit || c > maxLimit || d > maxLimit) return {error: "limit <= " + maxLimit };
  
  let sigDigi = (num, exp) => {
    while (num > 10) {
      num /= 10;
      exp++;
    }
    while (num < 1) {
      num *= 10;
      exp--;
    }
    return [num, exp];
  }
    
  let calcProb = (a, b, c, d) => {
    // prob = num * 10 ** exp;
    let num = 1;
    let exp = 0; 
    for (let i = 1;         i <= a;             i++) { [num, exp] = sigDigi(num / i, exp); }  // 1/a!
    for (let i = b + 1;     i <= a + b;         i++) { [num, exp] = sigDigi(num * i, exp); }  // (a+b)!/b!
    for (let i = c + 1;     i <= a + c;         i++) { [num, exp] = sigDigi(num * i, exp); }  // (a+c)!/c!
    for (let i = d + 1;     i <= c + d;         i++) { [num, exp] = sigDigi(num * i, exp); }  // (c+d)!/d!
    for (let i = b + d + 1; i <= a + b + c + d; i++) { [num, exp] = sigDigi(num / i, exp); }  // (b+d)!/(a+b+c+d)!
    return num * 10 ** exp;
  }

  let cutoffProb = calcProb(a, b, c, d);
  console.log(cutoffProb);
  
  let max = a + b;
  if (max > a + c) max = a + c;
 
  let pValue = 0;
  for (let i = 0; i <= max; i++) {
    let delta = a - i;
    let tmpProb = 1;
    if (b + delta >= 0 && c + delta >= 0 && d - delta >= 0) tmpProb = calcProb(i, b + delta, c + delta, d - delta);
    if (tmpProb <= cutoffProb) pValue += tmpProb;
  }
  if (pValue > 0.9999) pValue = 1; // 有効数字このくらい？
  return pValue;
}
```