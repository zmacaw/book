---
title: 堡垒机
---

开源和商业堡垒机知识和案例。

# jumpserver解密数据库密码

``` {.python}
python manage.py shell
from jumpserver.models import Setting
>>> s = Setting.objects.get()
>>> s.name
u'default'
>>> s.field1
u'ops'
>>> s.field2
u'20363'
>>> s.field3
u'16defe6a35805b61df3cd8e5b75b27697fc69fc7b0078a5ccc29a8988b66b9e8'
>>> s.field4
u'/opt/jumpserver-master/keys/default/admin_user.pem'
>>> s.field5
>>> from jumpserver.api import logger, Log, TtyLog, get_role_key, CRYPTOR, bash, get_tmp_dir
>>> CRYPTOR.decrypt(s.field3)
'password'
```
