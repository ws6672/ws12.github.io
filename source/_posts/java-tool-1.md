---
title: 自定义工具类探索之路（一）
date: 2019-10-26 16:13:38
tags: [java]
---


### 将数字年份转换为汉字年份

例如：2017 转换为 二〇一七
```
@Test
    public String yearnumToCn (Integer rq) {
        String[] hz = {"〇","一","二","三","四","五","六","七","八","九"};
        Integer index = 1000,tmp;
        StringBuilder stringBuilder = new StringBuilder();
        do {
            tmp = rq /index;
            stringBuilder.append(hz[tmp]);
            rq = rq % index;
            index /=10;
        } while (index >0);
        System.out.println(stringBuilder.toString());
        return stringBuilder.toString();
    }
```

##### 优化
