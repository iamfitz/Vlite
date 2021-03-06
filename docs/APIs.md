# __接口设计__

| method | url                                             | 描述                                             |
| :----- | :---------------------------------------------- | :----------------------------------------------- |
| POST   | user                                            | 通过表单提交啊，用户注册                         |
| GET    | user/notification                               | 获取用户通知信息                                 |
| GET    | user/todo                                       | 查看todo list                                    |
| GET    | user/(:user_name)                                 | 获取其他用户信息                                 |
| POST   | session                                         | 通过表单提交，用户登陆                           |
| GET    | session/user                                    | 获取自己的信息                                   |
| DELETE | session                                         | 用户退出登陆                                     |
| POST   | project                                         | 表单提交，新建项目                               |
| PUT    | project/(:project_id)                           | 通过表单提交，修改项目描述信息，标签，权限       |
| GET    | project/(:project_id)                           | 查看描述信息                                     |
| POST   | project/(:project_id)/file                      | 表单提交，上传文件                               |
| GET    | project/(:project_id)/file                      | 下载打包后的项目文件                             |
| POST   | project/(:project_id)/issue                     | 新建issue，表单提交                              |
| PUT    | project/(:project_id)/issue                     | 修改，关闭issue                                  |
| GET    | project/(:project_id)/issue                     | 列出project中的issue                             |
| GET    | project/(:project_id)/issue/(:issue_id)         | 列出project中的某个issue                         |
| GET    | issue?keyword=                                  | 搜索lable，milestone，title，open state搜索issue |
| GET    | project?name=&label=&owner=                     | 根据name，label，owner搜索project                |
| POST   | project/(:project_id)/issue/(:issue_id)/commnet | 回复issue                                        |
| POST   | project/(:project_id)/todo                      | 从issue中新建todo，隐式表单提交                  |

## POST:/user

功能:
通过表单提交，完成用户注册。

表单内容:
__user_name:str__ , __mail:str__ , __password:str__

返回内容:

```javascript
// 成功时
{
  "status":"success"，
  "message":""
}
// 失败时
{
  "status":"failed",
  "message":"some error message"
}
```

## GET:/user/notificatoin

功能；
通过session中储存的 __user_id__ 获取用户的被@的通知，自己的issue收到的回复，项目中新的issue。

注意:
被@的通知和自己issue的回复可能重复。

返回内容:

```javascript
{
  "at_notificatoin":
  [
    {
      "comment_id":213131
    },
    {
      "comment_id":213131
    }
  ],
  "issue_reply":
  [
    {
      "issue_id":0000,
      "reply_list":
      [
        {
          "commnet_id":02171
        }
      ]
    }
  ]
}
```

## GET:/user/todo

功能:
通过sesstion中储存的 __user_id__ 获取得到用户的todolist。

返回内容:

```javascript
[
  {
    "project_id":0000,
    "todolist":
    [
      {
        "todo_id":0000,
        "title":"todolist1",
        "content":"完成文档",
        "issue_id":0000
      },
      {
        "todo_id":0001,
        "title":"todolist2",
        "content":"完成页面",
        "issue_id":0001
      }
    ]
  }
]
```

## GET:/user/(:user_name)

功能:
通过 __url__ 中的 __user_id__ ，获取目标用户的信息。

返回内容:

```javascript
{
  "user_id":0001,
  "user_name":"test1",
  "user_email":"test1@test.com",
  "user_profile":"test account",
  "user_icon":"www.test.com/img/0001.png"
}
```

## POST:/session

功能:
通过表单提交，完成用户登陆。

表单内容为:

__user_mail(str)__ , __passwd(str)__

前端逻辑:
正则表达式判断是否为邮箱，是则直接使用，否则通过 GET /user/(:user\_name) 的返回结果，获取 user\_email。

后端逻辑:
进行登陆验证，成功时为 __session__ 中加入用户信息。(PHP中使用session是否需要手动的返回session_id?)

返回内容:

```javascript
// 成功时。返回json或者 redirect?
{
  "status":"success"，
  "message":""
}
// 失败时
{
  "status":"failed",
  "message":"some error message"
}
```

## POST:/project

功能:
通过表单提交，完成创建项目

表单内容为:

__project\_title(str)__ ， __project\_description(str)__ ， __project\_label(str)__ ，__file\_name(str)__ ，__version(str)__

前端逻辑:
将多个 __label__ 用 __+__ 连接成一个字符串(?)

后端逻辑：
通过 __POST__ 的内容，和 __session__ 中的信息，完成表插入的操作

返回内容:

```javascript
// 成功时。返回json或者 redirect?
{
  "status":"success"，
  "message":""
}
// 失败时
{
  "status":"failed",
  "message":"some error message"
}
```

## GET:/project/(:project_id)/issue

功能:
通过 __url__ 中的 __project\_id__ 获取其所有的 __issue__。

返回内容

```javascript
[h
  {
    "issue_id":0001,
    "issue_title":"test_issue1",
    "issue_description":"a test issue",
    "create_time":"2017-12-31 06:24:05",
    "close_time":"2017-12-31 06:24:06",
    "issue_lable":"test",
    "issue_statu":1
  }
]
```

## GET:/project/(:project\_id)/issue/(:issue\_id)

功能:
通过 __url__ 中的 __project\_id__ 和 __issue\_id__ 获取所有的 __comment__。

返回内容

```javascript
[
  {
    "comment_id":0001,
    "comment_content":"test_comment1",
    "comment_time":"2017-12-31 06:24:06",
    "user_name":"test1"
  }
]
```

## POST:/project/(:project\_id)/issue/(:issue\_id)/commnet

功能:
通过表单提交，完成回复issue。

表单内容为:
__issue\_content(str)__

后端逻辑：
通过 POST中的内容和__sesstion__ 中的用户信息，完成表插入.

返回内容:

```javascript
// 成功时
{
  "status":"success"，
  "message":""
}
// 失败时
{
  "status":"failed",
  "message":"some error message"
}
```

## POST:/project/(:project\_id)/todo

功能:
通过表单提交，完成从 __comment__ 创建 __todolist__。

表单内容:
__title(str)__ ， __content(str)__。

前端逻辑:
将 __issue__ 、 __comment__ 中的内容自动填入到 __content__ 中。