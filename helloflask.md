# Hello Flask Learning
```
from flask import Flask
app=Flask(__name__)
```
从flsak中导入Flask类，并通过实例化这个类，创建一个程序对象app

## 视图函数
处理某个请求的处理函数
可以作为某个路由的端点，用来生成URL
## 注册
给视图函数戴上一个装饰器
```
@app.route('/')
def hello():
	return 'Welcome to My Watchlist'
```
填入app.route()装饰器的第一个参数是URL规则字符串，当用户在浏览'/'这个URL的时候。就会触发这个函数
同样，参数也可以是'/home'
也可以在URL里定义变量部分，如下面的<name>即为变量：
```
@app.route('/user/<name>')
def user_page(name):
	return 'User Page'
```
## url_for函数
接受的第一个参数是端点值，默认为视图函数的名称
```
url_for('user_page',name='hewei')
```
输出是/user/hewei
## 模板
包含变量和运算逻辑的HTML或其他格式的文本叫做模板，执行这些变量替换和逻辑运算工作的过程被称为**渲染**
Flask会从程序实例所在模块同级目录的templates文件夹中寻找模板
### Jinja2模板渲染引擎
在模板里，需要添加特定的定界符将Jinja2语句和变量标记出来：
```
{{...}} 用来标记变量
{%...%} 用来标记语句，比如if语句，for语句
{#...#} 用来写注释
{{name|length}} 过滤器，用来获取name的长度
```
[Jinja2可用的过滤器](https://jinja.palletsprojects.com/en/2.10.x/templates/#list-of-builtin-filters)
### 使用render_template()函数可以把模板渲染出来
需要传入的参数为模板文件名（相对于templates根目录的文件路径），模板中所需要的变量值
如以下代码中，有index.html模板，需要name和movies两个变量
```
name=xxx
movies=xxx
def index():
	return render_template('index.html', name=name, movies=movies)
```
## 静态文件
不需要动态生成的文件，比如图片,css文件和JavaScripts脚本等
可以用url_for()函数来生成静态文件所在url
## 数据库
SQLite，基于文件，不需要单独启动数据库服务器，适合在开发时使用
### SQLAlchemy
可以通过定义Python类来表示数据库里的一张表，类属性表示表中的字段/列，称为模型类
```
db=SQLAlchemy(app) #初始化扩展，传入实例app
```
设置数据库URI，通过Flask.config字典配置
```
app.config['SQLALCHEMY_DATABASE_URI']='sqlite:///'+os.path.join(app.root_path,'data.dn')
#配置变量名称必须使用大写，加上数据库文件的绝对地址
```
### 创建数据库模型
```
class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(20))
```
模型类的继承为db.Model
每一个类属性要实例化db.Column，传入的参数为字段的类型
### 创建数据库表
```
import click
@app.cli.command() #将该方法变成一个命令行工具
@click.option('--drop', is_flag=True, help='Create after drop.')
#通过指定命令行选项的名称，从命令行读取参数值，再将其传递给函数
def initdb(drop):
	if drop:
		db.drop_all() #如果改动了模型类，想重新生成表模式，需要先使用db.drop_all删除表，然后重新创建
	db.create_all()
	click.echo('Initialized database.')
```
如此，便可以使用
$ flask initdb 命令创建数据库表
或是 $ flask initdb --drop 命令删除表后重建
### 数据库操作
user=User(name='Hewei') 创建一个User记录
db.session.add(user) 将新创建的记录添加到数据库会话
db.session.commit() 提交数据库会话

user=User.query.first() 读取User模型的第一个记录
*<模型类>.query.<过滤方法（可选）>.<查询方法>* 进行数据库读取操作
user=User.query.get(1) 获取id为1的记录
db.session.delete(user) 删除user记录
