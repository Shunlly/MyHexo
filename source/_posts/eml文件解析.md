---
title: eml文件解析
date: 2022-07-19 09:22:01
tags: mysql
categories: 学习
keywords: eml
description: eml文件解析
cover: https://typora-1303886849.cos.ap-guangzhou.myqcloud.com/typora%2F4327c995d22931a33f4ae872ee8605a42d8dc974.jpg%40942w_531h_progressive.jpg
top_img: https://typora-1303886849.cos.ap-guangzhou.myqcloud.com/typora%2F4327c995d22931a33f4ae872ee8605a42d8dc974.jpg%40942w_531h_progressive.jpg
---

# eml文件解析

eml文件要使用邮箱类的工具软件才能打开，可以用微软的outlook打开

```python
# -*- coding: utf-8 -*-

from flanker import mime
from datetime import datetime

def ParseEml(raw_email, path):
    with open(raw_email, 'rb') as fhdl:
        raw_email = fhdl.read()
        eml = mime.from_string(raw_email)
        # 主题
        subjece = eml.subject
        # 获取发信人地址及昵称
        eml_header_from = eml.headers.get('From')
        # 昵称
        # eml_from = address.parse(eml_header_from)
        # from_display_name = eml_from.display_name
        # 发送时间
        eml_time = eml.headers.get('Date')
        # if eml_time is not None:
        #     eml_time = eml_time.replace(',', '')
        #     dates = datetime.strptime(eml_time, '%a %d %b %Y %H:%M:%S %z')
        #     eml_time = dates.strftime("%Y-%m-%d %H:%M:%S")
        # 解析附件列表
        eml_attachs = ';'.join(x.detected_file_name for x in eml.parts if x.detected_file_name)
        if eml_attachs is not None and eml_attachs != '':
            attachs = eml_attachs.split(';')
            eml_attachs = '<p>附件</p>'
            for ats in attachs:
                eml_attachs += '<dl><dt><img src="../caseView/img/file_mail.png"></dt>' \
                               ' <dd title="OPPO_BR_20200707.log">' + ats + '</dd></dl>'
        # 内容
        eml_body = contentEml(eml)

        # 收件人信息
        eml_to = eml.headers.get('To')
        if eml_to is not None:
            eml_to = eml_to.replace('<', '&lt;').replace('>', '&gt;')

        #  eml_to_addr = address.parse_list(eml_to)
        # 多个地址以;拼接显示
        # if not type(eml_to_addr) == str:
        #     to_display_name = ';'.join(x.display_name for x in eml_to_addr if x.display_name)
        #     to_address = ';'.join(x.address for x in eml_to_addr if x.address)
   

# 邮件正文
def contentEml(eml):
    # 判断是否为单部分

    if eml.content_type.is_singlepart():
        eml_body = eml.body
    else:
        eml_body = ''
        for part in eml.parts:
            # 判断是否是多部分
            if part.content_type.is_multipart():
                eml_body, charset = contentEml(part)
            else:
                if part.content_type.main == 'text':
                    eml_body = part.body

    return eml_body

if __name__ == "__main__":
    ParseEml('aa.eml')
```

在eml解析这个环节中，我这边是借用了flanker这个第三方模块，这个模块提供了mine协议（eml基本上都是mine协议）的解析，在flanker中提供了将传进来的流文件转化为MIMEPart对象（这是flanker这个模块提供的），这个对象中包括了eml文件的信息（发送人、接收人、抄送人、发送时间、抄送时间、附件等基本信息），在对拿到的MIMEPart对象进行下一步处理，就得到了我们想要的信息。