# 接口自动化平台用例执行引擎 — ApiTestEngine

## 前言

​		   ApiTestEngine主要是为接口自动化平台 开发的一个Http接口用例执行的引擎，其实之前开发的接口自动化框架apin也可以作为测试平台的用例执行引擎。但是apin最初设计的初衷是基于json或yaml文件来编写测试用例，里面设计了更多规则。用来进行平台开发反而有些笨重了，也不适合在课程中教学适用，于是在apin的基础写进行了精简和优化，开发了 ApiTestEngine这个更为精简和灵活的用例执行引擎。下面介绍一下ApiTestEngine的具体使用。

安装命令:
   ```python
pip install ApiTestEngine
``` 
    	

## 一、用例执行函数

- 用例执行入口函数为core.cases模块中的run_test，具体使用如下

> #####  run_test参数说明
>
> - ##### case_data: 执行的测试数据
>
> - ##### env_config：执行的环境配置
>
> - ##### debug：是否为debug模式(单接口调试运行使用debug模式)

```python
# 测试数据(详细结构说明看下一节)
case_data = [{},{}]
# 运行环境数据(详细结构说明看第三节)
config = {}
result = run_test(case_data=case_data, env_config=config, debug=False)
```



## 二、测试数据结构

下面详细介绍run_test执行测试的用例数据(case_data参数)的‘’

#### 1、 测试数据基本结构

> ```python
> [
>   {
>   'name':"测试场景名称1"
>   'cases':['用例数据1','用例数据2']
>   },		
>   {
>   'name':"测试场景名称2"
>   'cases':['用例数据3','用例数据4']
>   },
> 	......
> ]
> ```
>
> 执行的用例数据为list类型，列表中的元素为测试场景(dict类型)，测试场景有两个字段，name和cases
>
> - ###### name:测试场景的名称
>
> - ###### cases:包含该测试场景下所有用例的列表

#### 2、测试场景中测试用例的数据结构

> 注意：测试用例数据为一个字典,具体结构如下：
>
> ```python
> {
> "title": "测试用例2",
> "host": "http://httpbin.org/post",
> "interface": {
>  "url": "/post",
>  "name": "登录",
>  "method": "post",
>  },
> "headers": {
> 	'content-Type': "application/json"
> 	},
> "request": {
>  'json': {"mobile_phone": "${{user_mobile}}","pwd": "lemonban"},
> 	},
> 'setup_script': "print('前置脚本123')",
> 'teardown_script': "test.assertion('相等',200,response.status_code)"
> }
> ```
>
> ##### 用例字段说明：
>
> - ##### title: 用例名称( 必填)
>
> - ##### host: 测试接口的host地址 (非必填，如果没填则会使用全局的变量中的host)
>
> - ##### interface：请求接口，字典类型(必填),包含三个字段：
>
>     - ###### name: 接口名(非必填)
>
>     - ##### url: 接口路径（必填）
>
>     - ##### method：请求方法(必填)
>
> - ###### headers：请求头，字典类型(非必填，如果全局遍历中设置了请求头，则会将此处请求头和全局遍历请求头合并)
>
> - ##### request：请求参数，字典类型(非必填，支持requests模块请求的所有字段)，常用字段如下
>
>     - ###### data:传递表单参数(同requests模块的data)
>
>     - ###### json:传递json参数(同requests模块的json)
>
>     - ###### params:传递查询参数(同requests模块的params)
>
>     - ###### files:上传文件,参数具体格式如下：
>
>      ```python
>      # { "参数名": ["文件名", "文件路径", "文件类型"]}
>      {
>          "name": ["19.png", r"./image/19.png", "image/png"]
>      }
>      ```
>
>      提示： 其他requests模块中支持的字段这里都支持
>
> - ###### setup_script：用例前置脚本,字符串类型(具体使用参照第五节)
>
> - ###### teardown_script：用例后置脚本，字符串类型(具体使用参照第五节)

#### 3、用例中引用全局遍历

在测试数据的header、interface,requests中可以引用遍历

> ##### 变量引用语法：${{变量名}}
>
> ```python
> "request": {
>  'json': {"mobile_phone": "${{user_mobile}}","pwd": "lemonban"},
> 	}
> # 说明：上面requests的json参数中引用了变量 user_mobile
> ```



## 三、环境数据说明

环境数据一共包含三个字段：ENV，DB，global_func

> ##### 1、ENV :  全局变量，字典类型(必填项)，存放全局的host，自定义的全局变量，
>
> ##### 2、DB：数据库配置对象，列表类型(必填项)，支持同时连接多个数据库服务，每个连接为列表中的一个元素，连接配置信息如下
>
> ```python
> {
> "name": "aliyun",
> "type": "mysql",
> "config": {
>  "user": "future",
>  "password": "123456",
>  "host": "api.lemonban.com",
>  "port": 3306
>  }
> }
> ```
>
> - ###### name: 连接名(字符串，只能用英文)，在用例脚本可以通过db.连接名.去执行sql语句(参照前后置脚本)
>
> - ##### type:数据库类型(目前仅支持mysql数据库，可以自己扩展开发)
>
> - ##### config:数据库连接信息，字典类型，字段如下
>
>     - user ：账号
>     - password： 密码
>     - host: 数据库的host地址
>     - port：数据库的端口
>
> #### 3、global_func：全局工具函数(字符串)
>
> - 建议：在进行平台在开发时，将global_func设置问大文本字段，前端编辑完，后端直接存在数据库，执行用例时读取出来传入即可



## 四、全局工具函数说明

global_func中定义的全局函数，在用例的前后置脚本中可以直接调用，调用方式如下

![1651145768956](readme.assets/1651145768956.png) 



## 五、前后置脚本

为了让用户 更方便的去编写前后置脚本进行测试，框架本身也预设了一些变量和一些方法

#### 1、脚本中预置的对象：

> ##### ENV:全局变量
>
> ##### env：临时变量
>
> ##### response:请求响应对象(后置脚本可以访问)

#### 2、脚本中预置的方法

> ###### test.re_extract: 正则提取数据
>
> ​    参数1：数据源（str类型）
> ​    参数2：提取表达式
>
> ###### test.json_extract：jsonpath提取数据
>
> ​    参数1：数据源(dict类型 or list类型)
> ​    参数2：提取表达式
>
> ###### test.save_global_variable:设置全局变量
>
> ​    参数1：变量名
> ​    参数2：变量值
>
> ###### test.save_env_variable：设置临时变量
>
> ​    参数1：变量名
> ​    参数2：变量值
>
> ###### test.delete_global_variable:删除全局变量
>
> 	参数：变量名
>
> ###### test.delete_env_variable:删除临时变量
>
> ​    参数：变量名
>
> ##### test.assertion:断言方法
>
> ​    参数1：断言方法
> ​    参数2：预期结果
> ​    参数2：实际结果

如果以上内置的方法不能满足需求，可在全局的工具函数中定义相关的方法，在脚本中调用。

关于ApiTestEngine的基本使用就给大家介绍到这里啦，大家可以去研究一下，用来编写测试平台

