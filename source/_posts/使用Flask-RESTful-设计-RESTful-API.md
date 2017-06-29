---
title: "使用Flask-RESTful-设计-RESTful-API"
date: 2017.05.09 20:55
tags:
 - Flask
 - RESTful
---

![restful](http://upload-images.jianshu.io/upload_images/3340699-2d0059d7e5c36496.gif?imageMogr2/auto-orient/strip)Flask-RESTful 提供了一个Resource基础类，能够定义一个给定URL的一个或者多个HTTP方法。

------------------------

上一篇TODO list应用改用Flask_RESTful的完整示例代码：
```
from flask import Flask, abort, url_for, make_response, jsonify
from flask_restful import Resource, Api, reqparse, fields, marshal
from flask_httpauth import HTTPBasicAuth

app = Flask(__name__)
api = Api(app)
auth = HTTPBasicAuth()

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

task_fields = {
	'title': fields.String,
	'description': fields.String,
	'done': fields.Boolean,
	'uri': fields.Url('task')
}

class TaskListAPI(Resource):
	decorators = [auth.login_required]
	def __init__(self):
		self.reqparse = reqparse.RequestParser()
		self.reqparse.add_argument('title', type=str, required=True,
			help='No task title provided', location='json')
		self.reqparse.add_argument('description', type=str, default="", 
			location='json')
		self.reqparse.add_argument('done', type=bool, location='json')
		super(TaskListAPI, self).__init__()

	def get(self):
		return jsonify(list(map(marshal, tasks, [task_fields for i in range(len(tasks))])))

	def post(self):
		task = {}
		args = self.reqparse.parse_args()
		task['id'] = tasks[-1]['id']+1
		for k, v in args.items():
			if v != None:
				task[k] = v
		tasks.append(task)
		return {'task':task}, 201

class TaskAPI(Resource):
	decorators = [auth.login_required]
	def __init__(self):
		self.reqparse = reqparse.RequestParser()
		self.reqparse.add_argument('title', type=str, location='json')
		self.reqparse.add_argument('description', type=str, location='json')
		self.reqparse.add_argument('done', type=bool, location='json')
		super(TaskAPI, self).__init__()

	def get(self, id):
		task = list(filter(lambda x: x['id']==id, tasks))
		if len(task)==0:
			abort(404)
		task = task[0]
		return {'task': marshal(task, task_fields)}

	def put(self, id):
		task = list(filter(lambda t: t['id'] == id, tasks))
		if len(task) == 0:
			abort(404)
		task = task[0]
		args = self.reqparse.parse_args()
		for k, v in args.items():
			if v != None:
				task[k] = v
		# return jsonify(task=make_public_task(task))
		return {'task': marshal(task, task_fields)}

	def delete(self, id):
		task = list(filter(lambda x: x['id']==id, tasks))
		if len(task)==0:
			abort(404)
		task = task[0]
		tasks.remove(task)
		return {'result':True}

api.add_resource(TaskListAPI, '/todo/api/v1.0/tasks', endpoint='tasks')
api.add_resource(TaskAPI, '/todo/api/v1.0/tasks/<int:id>', endpoint='task')


def make_public_task(task):
	new_tasks = {}
	for field in task:
		if field=='id':
			new_tasks['uri'] = url_for('task', id=task['id'], _external=True)
		else:
			new_tasks[field] = task[field]
	return new_tasks


@auth.get_password
def get_password(username):
	if username=='hukx':
		return '123456'
	return None

@auth.error_handler
def unauthorized():
	# return make_response(jsonify(error='Unauthorized access'), 401)
	return make_response(jsonify(error='Unauthorized access'), 403)



if __name__ == '__main__':
	app.run(debug=True)


```
原文链接：http://www.pythondoc.com/flask-restful/second.html
