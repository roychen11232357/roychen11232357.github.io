---
title: GSON反序列化時int被轉成float的問題
date: 2022-10-05 16:51:48
tags:
- java
---
GSON是一套Google發佈的Java對json序列化/反序列化的lib, java開發者應該都不陌生, 但是可能會遇到int被轉成float的問題, 

例如, 原始的json是

```
{
  "test": 1
}
```
GSON轉完變成
```
test: 1.0
```
這可能會造成一些非預期的錯誤, 我們預期的結果, 當1轉完還是1, 1.5轉完還是1.5, 

一直以來GSON處理這個問題都是要寫一些type assertion的處理, 例如

```
if(unTypeVal instanceof Number) 
...
...
if(doubleNum == Math.round(doubleNum))
```

最終這個問題在2021年發佈的2.8.9版本中有了乾淨解法, 

可以看下面這段code, 初始化gson object時, 可以指定反序列化策略, LONG_OR_DOUBLE顧名思義就是要區分整數或浮點數, 如此一來, 代碼變得超乾淨!

```
Gson gson = new GsonBuilder()
        .setNumberToNumberStrategy(ToNumberPolicy.LONG_OR_DOUBLE)
        .setObjectToNumberStrategy(ToNumberPolicy.LONG_OR_DOUBLE)
        .create();
```