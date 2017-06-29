---
title: "RESTful API的Flask手动实现"
date: 2017.05.09 16:02
tags:
 - RESTful
 - Flask
---

![](http://upload-images.jianshu.io/upload_images/3340699-9825d956da07f0b6.gif?imageMogr2/auto-orient/strip)RESTful web services 概念的核心就是“资源”。资源可以用URI来表示。客户端使用HTTP协议定义的方法来发送请求到这些URIs，导致这些被访问的资源状态改变。

-----------------------

### 模式
- GET  http://example.com/api/orders  获取资源的信息
- GET  http://example.com/api/orders/123 获取某个特定资源的信息
- POST  http://example.com/api/orders 创建新资源 
- PUT http://example.com/api/orders/123 更新资源
- DELETE http:/example.com/api/orders/123 删除资源

------------------------

### 举例
创建一个TODO list应用，设计URL模式为： `http://[hostname]/todo/api/v1.0/`

HTTP方法  |  URL   |   动作
-----------|-----------------------|---------------
GET  |  http://[hostname]/todo/api/v1.0/tasks  |  检索任务列表
GET  |  http://[hostname]/todo/api/v1.0/tasks/[task_id] |  检索某个任务
POST |  http://[hostname]/todo/api/v1.0/tasks  |  创建新任务
PUT |  http://[hostname]/todo/api/v1.0/tasks/[task_id]  |  更新任务
DELETE |  http://[hostname]/todo/api/v1.0/tasks/[task_id] | 删除任务


任务属性：id(数字)、title(字符串)、description(具体描述、文本)、done(任务完成状态、布尔)

代码：
```
from flask import Flask, jsonify, abort, make_response, request, url_for

app = Flask(__name__)

tasks = [
	{
		'id':1,
		'title':'白鹿原',
		'description':'这是一定死啦塞啊；阿凯骄傲灵丹安矿鉴定暗访东方三；啊反馈；安静快撒娇扥卡机',
		'done':False
	},
	{
		'id':2,
		'title':'嫌疑人',
		'description':'as大肯借；额看课文静安寺放大反抗航将；来否；付定金奥兰多 看企鹅长恨东澳岛啊；伐开森',
		'done':False
	}
]

@app.route('/')
def index():
	return 'Hello world'

@app.route('/todo/api/v1.0/tasks', methods=['GET'])
def get_tasks():
	return jsonify({'tasks':list(map(make_public_task, tasks))})

@app.route('/todo/api/v1.0/tasks/<int:task_id>', methods=['GET'])
def get_task(task_id):
	task = list(filter(lambda x: x['id']==task_id, tasks))
	if len(task)==0:
		abort(404)
	return jsonify(task=make_public_task(task[0]))

@app.route('/todo/api/v1.0/tasks', methods=['POST'])
def create_task():
	if not request.json or 'title' not in request.json:
		abort(400)
	task = {
		'id':tasks[-1]['id']+1,
		'title': request.json['title'],
		'description':request.json.get('description',''),
		'done':False
	}
	tasks.append(task)
	return jsonify(task=task), 201

@app.route('/todo/api/v1.0/tasks/<int:task_id>', methods=['PUT'])
def update_task(task_id):
	task = list(filter(lambda x: x['id']==task_id, tasks))
	if len(task)==0:
		abort(404)
	if not request.json:
		abort(400)
	# 验证各个字段...
	if 'title' in request.json and type(request.json['title'])!= str:
		abort(400)
	if 'description' in request.json and type(request.json['description']) is not str:
		abort(400)
	if 'done' in request.json and type(request.json['done']) is not bool:
		abort(400)
	task[0]['title'] = request.json.get('title', task[0]['title'])
	task[0]['description'] = request.json.get('description', task[0]['description'])
	task[0]['done'] = request.json.get('done', task[0]['done'])
	return jsonify(task=task[0])

@app.route('/todo/api/v1.0/tasks/<int:task_id>', methods=['DELETE'])
def delete_task(task_id):
	task = list(filter(lambda x: x['id']==task_id, tasks))
	if len(task)==0:
		abort(404)
	tasks.remove(task[0])
	return jsonify(result=True)

def make_public_task(task):
	# print('😲',task)
	new_tasks = {}
	for field in task:
		if field=='id':
			new_tasks['uri'] = url_for('get_task', task_id=task['id'], _external=True)
		else:
			new_tasks[field] = task[field]
	return new_tasks


@app.errorhandler(404)
def not_found(error):
	return make_response(jsonify(error='Not Found'), 404)

@app.errorhandler(400)
def bad_request(error):
	return make_response(jsonify(error='Bad Request'), 400)

if __name__ == '__main__':
	app.run(debug=True)

```
原文链接：http://www.pythondoc.com/flask-restful/first.html
