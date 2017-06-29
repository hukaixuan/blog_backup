---
title: "Flask request 模块概览"
date: 2017.04.10 11:25
tags:
 - Flask
---

---------------
### request.form   
- 获取POST或PUT的数据, `request.form['data_of_post_or_put']`
- 为防止 `request.form['key']` 因key不存在抛出异常，最好用 `request.form.get('key', 'default')`

### request.args
- 获取通过URL传递的参数(?key=value)
- `search_word = request.args.get('key', '') `

### request.files
- 记得在上传文件的form中添加属性：`enctype="multipart/form-data"`
- 通过 `f = request.files['the_file']` 获取暂存在内存或缓存中的文件，
    然后通过调用 `f.save('/the/path/you/want/to/save/and/the/new/filename')`
- 如果你想要获取原文件的文件名(`f.filename`)进行保存，切记用户上传的文件名可能是不安全的，可以 `from werkzeug.utils import secure_filename` , 通过 `f.save('/path/' + secure_filename(f.filename))`(注：`secure_filename ` 会过滤掉中文文件名)

### request.cookies
- 通过 response 设置 cookies 
`from flask import make_response`
`resp = make_response(render_template(...))`
`resp.set_cookie('username', 'the username')`
- 通过 request 获得 cookies `request.cookies.get('username')`
