### blueprint

- 使用

  ```python
  from flask import Blueprint
  #可以对蓝本加前缀
  user  = Blueprint("user", __name__, url_prefix="/user")
  
  @user.route("/login/")
  def login():
      return "欢迎登陆"
  @user.route("/register/")
  def register():
      return "欢迎注册"
  ```

- 在register_blueprin函数中也可以使用url_prefix参数

  ```python
  from flask import Flask
  from flask_script import Manager
  from user import user
  
  app = Flask(__name__)
  manager  = Manager(app)
  #register_blueprint函数可以加上前缀的，
  #他具有更高的优先级，，但是不用，你的前缀一般写在蓝本中
  app.register_blueprint(user, url_prefix="/u")
  if __name__ == "__main__":
      manager.run()
  ```

### cookie（重要）面试必问

- http状态是一个无状态的连接，因为用户在点击网站的时候，网站是不会记录你的上一次点击状态的行为

  必须有cookie来保持他的状态。这样用户在点击下一个操作的时候就不用再次登录，用户的体验是相当好的

  当第一次去访问网页的时候，会产生一个cookie，并携带，每次去访问同一个网站的时候都会携带

  <https://www.cnblogs.com/wangjintao-0623/p/9598827.html> 

  ```python
  from flask import Blueprint,request, make_response
  
  cookie = Blueprint("cookie", __name__)
  
  #得到cookie
  @cookie.route("/get/")
  def get_cookie():
      #如果name的值为空的话，name的值就是我是谁？？？
      #name有值的话，就返回的是真实cookie值
      return request.cookies.get("name", "我是谁？？？")
  
  #设置cookie
  @cookie.route("/set/")
  def set_cookie():
      resp = make_response("cookie已设置")
      #过期时间
      #max_age:有效期，是一个整数，是一个秒数
      #expirs:有效期，是一个datetime时间的类型的数据，但是不用
      resp.set_cookie("name", "qqqq", max_age=10)
      return resp
  #销毁cookie
  @cookie.route("/del/")
  def del_cookie():
      resp = make_response("cookie已销毁")
      resp.delete_cookie("name")
      return resp
  
  ```

### session(会话)

- 说明：session是比cookie更加安全的一种携带数据的方式，功能都是一样的额，一般都使用session

- session会把数据存到服务器中，cookie存到客户端里

  ```python
  from flask import  Blueprint,session
  
  sess = Blueprint("sess", __name__, url_prefix="/sess")
  #获取session
  @sess.route("/get/")
  def get_sess():
      return session.get("name", "who are you")
  #设置session
  @sess.route("/set/")
  def set_sess():
      #s设置永久有效,默认是false，只要关闭了浏览器，sesssion就失效了
      #可以在这个地方设置，也可以在app里面设置
      session.permanent = True
      session["name"] = "zxcv"
      return "session已设置"
  #删除session
  @sess.route("/del/")
  def del_sess():
      session.clear()
      return "session已删除"
  ```

### 模板引擎（了解）

- 说明：模板文件，就是按照一定规则书写前端的（html）代码的，模板引擎就是提供替换和解析的工具
- 是flask核心人员开发的（jinja2）

（注意：下周我会给大家讲vue前端框架，以后就不用了）

使用：

- 模板渲染：

  - 在templates文件夹下面写咱们的模板文件，

  - 在试图函数中写咱们渲染，render_template()

  - 还可以使用一个叫render_template_string()(很少用)

    ```python
    #S设置模板文件的自动加载
    app.config["TEMPLATES_AUTO_RELOAD"] = True
    @app.route("/")
    def index():
        #如果你想要使用模板文件的话，必须使用模板渲染
        return render_template("index.html")
        #return render_template_string("<h1>呵呵哒，傻逼了吧</h1>")
    
    ```

    

- 变量的使用

  - 在模板文件中加载变量的时候都必须加{{}}

  - 在render_template()函数中传递变量的时候，可以写字典

    ```python
    @app.route("/var/")
    def var():
        #将name变量给前端页面分配过去了。
       # g.name = "ergou1"
        return render_template("var.html",student={"name":"gouer", "age":"12"})
    
    ```

- 

- 使用过滤器

  - upper:转换成大写

  - lower：全部小写

  - title：每个单词的首字母大写

  - capitalize:首字母大写

  - trim：去掉两边的空白

    <https://www.cnblogs.com/baijinshuo/p/10245418.html> 

### 文件上传

```python
from flask import Flask, request, render_template, url_for, send_from_directory
from flask_script import Manager
import os

app = Flask(__name__)
manager = Manager(app)
app.config["UPLOAD_FOLDER"] = os.path.join(os.getcwd(), "static/upload")

#图片显示的路由
@app.route("/upload/<filename>")

def uploaded(filename):
    return send_from_directory(app.config["UPLOAD_FOLDER"], filename=filename)
@app.route("/upload/", methods=["POST", "GET"])
def upload():
    img_url = None
    if request.method == "POST":
        #获取上传图像的对象
        photo = request.files.get("photo")
        print(photo)
        #meinv.jpg

        print(photo.filename)
        if photo:
            #保存起来，
            pathname= os.path.join(app.config["UPLOAD_FOLDER"], photo.filename)
            #b保存上传文件
            photo.save(pathname)
            #构造这个img_url
            img_url = url_for("uploaded", filename = photo.filename)
            print(img_url)
        else:
            return "上传失败"

    return render_template("upload.html", img_url=img_url)
if __name__ == "__main__":
    manager.run()
    
    hsjsjsjm.nsjasas.jpg
```

- 邮件发送

  ```python
  from flask import Flask
  from flask_script import Manager
  #pip install flask-mail
  from flask_mail import Mail,Message
  import os
  
  
  app = Flask(__name__)
  manager = Manager(app)
  
  #邮件发送需要配置一些东西
  app.config["MAIL_SERVER"] = "smtp.1000phone.com"
  #用户名
  app.config["MAIL_USERNAME"] = "wangbo3@1000phone.com"
  #密码
  app.config["MAIL_PASSWORD"] = os.getenv("MAIL_PASSWORD", "123456")
  #实例化发送邮件对象
  mail = Mail(app)
  @app.route("/send/")
  def send_mail():
      msg = Message("账户激活",recipients=["1486728869@qq.com"], sender=app.config["MAIL_USERNAME"])
      msg.html="恭喜你中奖了"
      mail.send(msg)
      return "邮件已发送"
  
  if __name__ == "__main__":
      manager.run()
  ```

  





