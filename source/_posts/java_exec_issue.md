---
title: java使用runtime exec調用python程式過慢的解決方式 (改用http)
date: 2022-10-04 16:02:27
tags: 
- performance
- java
- python
---
某專案需要在java調用python的一個小程式做一個複雜的字串解析, 由於該解析的lib在其他語言並沒有現成的, 所以先使用系統呼叫, 透過stdout來拿到結果

`Runtime.getRuntime().exec(cliStr)`

但這樣會有效能問題, 我發現當cpu core數不夠, 且遇到高併發的場景時, 該系統呼叫會變得很慢(接近1s), 

查了一下大概有幾種解法

- 使用Jython, 基於jvm上執行python
- 起一個輕量python webserver (ex. flask)來專門處理該解析工作

先講Jython這招吧, 是有試出一個hello world的小實驗, 但卡在目前似乎只支援python2, 且如果python程式需要相依其他python module時, 不知道該如何處理, 所以這招就先不採用

最後採用http調用python webserver(flask)的方式拿到我們需要的結果, 用此方式可以大大加快系統呼叫的耗時, 我們的情境大概可以快500-700ms (1core, 10併發的複雜字串處理邏輯),

只是起一個額外的flask需要再處理一些容錯機制, 例如, flask掛掉怎麼辦? 我的做法分兩個面向

- flask api本身邏輯會用try cache捕捉任何錯誤, 回傳空字串
- 萬一是網路或資源或任何非預期錯誤時導致調用api失敗時, java這裡會去用原本效能較慢的系統呼叫方式拿到結果, 此外, 會再用系統呼叫嘗試啟動flask, 當flask已經掛了就可以復原, 但flask還活著時, 也會因port已經被占用而直接跳掉啟動程序


