# 20180106 Datetime和String转换

今日谈谈python对于字符串和日期对象的互相转换问题。

## String转datetime

参考：https://stackoverflow.com/questions/466345/converting-string-into-datetime

```
from datetime import datetime
datetime_object = datetime.strptime('Jun 1 2005  1:33PM', '%b %d %Y %I:%M%p')
```

## Datetime转string

参考：https://stackoverflow.com/questions/10624937/convert-datetime-object-to-a-string-of-date-only-in-python

```
import datetime
t = datetime.datetime(2012, 2, 23, 0, 0)
t.strftime('%m/%d/%Y')

```