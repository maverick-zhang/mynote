# flask-RESTful Request

### 1. Request parser

- ​	先定义一个RequestParser对象，通过add_argument("argu_name", type, help=xxx)对传入的query_params进行预校验：参数是否必须提供（required=True/False）,类型验证，错误提示（help），当一个key存在多个value时，是否获取全部（action="append"), dest=xxx指定别名， location=xxx指定arg获取的位置可以选择在form表单中获取或者args中获取(即query_params)以及headerds和cookie中获取到信息，可以同时指定多个位置，以列表的形式作为lacation的val，默认的位置为form。

  在验证之后，通过RequestParser对象的parse_args()获取对应的参数，可以使用["arg_name"]进行索引，也可以使用get("arg_name")

```python
parser = reqparse.RequestParser()
parser.add_argument("g_name", type=str,required=True ,help="please input g_name")
parser.add_argument("g_price", type=float, help="please input number")
parser.add_argument("mu", action="append")
parser.add_argument('rname', dest="name")
parser.add_argument("OUTFOX_SEARCH_USER_ID_NCOO", dest="lo", action="append", location=["cookies", "args"])
parser.add_argument("User-Agent",dest="ua", location="headers")


class GoodsListResource(Resource):

    # @marshal_with(multi_goods_fields)
    def get(self):

        args = parser.parse_args()

        print(args.get("lo"))

        print(args.get("ua"))
```

​	RequestParser可以通过copy函数进行继承

```python
from flask_restful import reqparse

parser = reqparse.RequestParser()
parser.add_argument('foo', type=int)

parser_copy = parser.copy()
parser_copy.add_argument('bar', type=int)
```