Flasgger
Flask API的Easy Swagger UI
构建状态 代码健康 测试覆盖状态 PyPI 通过Paypal捐助

flasgger

Flasgger是一个Flask扩展，可从您的API中所有已注册的Flask视图中提取OpenAPI-Specification（以下简称"spec"）。

Flasgger还集成 SwaggerUI，因此您可以访问http://localhost:5000/apidocs并可视化并与您的API资源进行交互。

Flasgger还使用与可以验证以POST，PUT，PATCH形式接收的数据是否与YAML，Python字典，Marshmallow Schema所定义的spec一致。

Flasgger可以使用简单的函数视图或方法视图（使用docstring作为规范），或使用@swag_from装饰器从YAML或dict获取spec，还提供可以使用SwaggerView调用Marshmallow Schemas作为spec。

Flasgger与Flask-RESTful兼容，因此您可以同时使用Resources和swag spec，看看restful示例。

Flasgger还支持将Marshmallow APISpec作为规范的spec模板，如果您使用的是Marshmallow的APISPec，请查看apispec示例。

高度参与的贡献者


示例和演示应用
有一些示例应用程序 ，您也可以在Flasgger演示应用程序中使用示例。

注意：所有示例应用程序也是测试用例，并在Travis CI中自动运行以确保质量和测试覆盖范围。

Docker
示例和演示应用程序也可以作为Docker映像/容器构建和运行：

docker build -t flasgger .
docker run -it --rm -p 5000:5000 --name flasgger flasgger
然后访问位于 http://localhost:5000 的Flasgger演示应用程序。

安装
在您的virtualenv下执行以下操作：

确保您拥有最新的setuptools

pip install -U setuptools
然后

pip install flasgger
或（开发版本）

pip install https://github.com/flasgger/flasgger/tarball/master
注意：如果要使用Marshmallow Schemas，则还需要运行pip install marshmallow apispec。

入门
使用文档字符串作为spec
创建一个名为colors.py的文件

from flask import Flask, jsonify
from flasgger import Swagger

app = Flask(__name__)
swagger = Swagger(app)

@app.route('/colors/<palette>/')
def colors(palette):
    """示例端点按调色板返回颜色列表
    这是使用文档字符串作为spec。
    ---
    parameters:
      - name: palette
        in: path
        type: string
        enum: ['all', 'rgb', 'cmyk']
        required: true
        default: all
    definitions:
      Palette:
        type: object
        properties:
          palette_name:
            type: array
            items:
              $ref: '#/definitions/Color'
      Color:
        type: string
    responses:
      200:
        description: 颜色列表（可被调色板过滤）
        schema:
          $ref: '#/definitions/Palette'
        examples:
          rgb: ['red', 'green', 'blue']
    """
    all_colors = {
        'cmyk': ['cyan', 'magenta', 'yellow', 'black'],
        'rgb': ['red', 'green', 'blue']
    }
    if palette == 'all':
        result = all_colors
    else:
        result = {palette: all_colors.get(palette)}

    return jsonify(result)

app.run(debug=True)
现在运行：

python colors.py
并转到：http://localhost:5000/apidocs/

您应该得到：

colors

使用外部YAML文件
保存一个新文件colors.yml

示例端点按调色板返回颜色列表
在此示例中，规范取自外部YAML文件
---
parameters:
  - name: palette
    in: path
    type: string
    enum: ['all', 'rgb', 'cmyk']
    required: true
    default: all
definitions:
  Palette:
    type: object
    properties:
      palette_name:
        type: array
        items:
          $ref: '#/definitions/Color'
  Color:
    type: string
responses:
  200:
    description: 颜色列表（可被调色板过滤）
    schema:
      $ref: '#/definitions/Palette'
    examples:
      rgb: ['red', 'green', 'blue']
让我们使用相同的示例，仅更改视图功能。

from flasgger import swag_from

@app.route('/colors/<palette>/')
@swag_from('colors.yml')
def colors(palette):
    ...
如果您不想使用装饰器，则可以使用docstringfile:快捷方式。

@app.route('/colors/<palette>/')
def colors(palette):
    """
    file: colors.yml
    """
    ...
使用字典作为raw spec
创建一个Python字典为：

specs_dict = {
  "parameters": [
    {
      "name": "palette",
      "in": "path",
      "type": "string",
      "enum": [
        "all",
        "rgb",
        "cmyk"
      ],
      "required": "true",
      "default": "all"
    }
  ],
  "definitions": {
    "Palette": {
      "type": "object",
      "properties": {
        "palette_name": {
          "type": "array",
          "items": {
            "$ref": "#/definitions/Color"
          }
        }
      }
    },
    "Color": {
      "type": "string"
    }
  },
  "responses": {
    "200": {
      "description": "颜色列表（可被调色板过滤）",
      "schema": {
        "$ref": "#/definitions/Palette"
      },
      "examples": {
        "rgb": [
          "red",
          "green",
          "blue"
        ]
      }
    }
  }
}
现在为同一个函数使用dict替代YAML文件。

@app.route('/colors/<palette>/')
@swag_from(specs_dict)
def colors(palette):
    """Example endpoint returning a list of colors by palette
    In this example the specification is taken from specs_dict
    """
    ...
使用 Marshmallow Schemas
首先： pip install marshmallow apispec

from flask import Flask, jsonify
from flasgger import Swagger, SwaggerView, Schema, fields


class Color(Schema):
    name = fields.Str()

class Palette(Schema):
    pallete_name = fields.Str()
    colors = fields.Nested(Color, many=True)

class PaletteView(SwaggerView):
    parameters = [
        {
            "name": "palette",
            "in": "path",
            "type": "string",
            "enum": ["all", "rgb", "cmyk"],
            "required": True,
            "default": "all"
        }
    ]
    responses = {
        200: {
            "description": "颜色列表（可被调色板过滤）",
            "schema": Palette
        }
    }

    def get(self, palette):
        """
        Colors API using schema
        This example is using marshmallow schemas
        """
        all_colors = {
            'cmyk': ['cyan', 'magenta', 'yellow', 'black'],
            'rgb': ['red', 'green', 'blue']
        }
        if palette == 'all':
            result = all_colors
        else:
            result = {palette: all_colors.get(palette)}
        return jsonify(result)

app = Flask(__name__)
swagger = Swagger(app)

app.add_url_rule(
    '/colors/<palette>',
    view_func=PaletteView.as_view('colors'),
    methods=['GET']
)

app.run(debug=True)
注意：请查看examples/validation.py，以获得更完整的示例。

注意：在路径规则(path rule)中捕获参数时，请始终使用显式类型，不可以：/api/<username> 可以：/api/<string:username>

使用 Flask RESTful 资源 （Resources）
Flasgger与Flask-RESTful兼容，您只需要安装pip install flask-restful，然后：

from flask import Flask
from flasgger import Swagger
from flask_restful import Api, Resource

app = Flask(__name__)
api = Api(app)
swagger = Swagger(app)

class Username(Resource):
    def get(self, username):
       """
       本示例使用FlaskRESTful资源
       它也适用于swag_from，schema和spec_dict
       ---
       parameters:
         - in: path
           name: username
           type: string
           required: true
       responses:
         200:
           description: 单个用户项
           schema:
             id: User
             properties:
               username:
                 type: string
                 description: 用户名
                 default: 匿名用户
        """
        return {'username': username}, 200


api.add_resource(Username, '/username/<username>')

app.run(debug=True)
自动解析外部YAML文档和 MethodViews
可以将Flasgger配置为自动解析外部YAML API文档。在您的 app.config['SWAGGER']中设置Set a doc_dir，然后Swagger将加载API文档通过在doc_dir中查找由端点名称和函数名称存储的YAML文件。例如，'doc_dir': './examples/docs/'和文件./examples/docs/items/get.yml将为ItemsView方法get提供Swagger文档。

另外，当如上使用Flask RESTful时，通过在构造Swagger时传递parse=True，Flasgger将使用flask_restful.reqparse.RequestParser，找到所有MethodViews，然后将解析和验证的数据存储在flask.request.parsed_data中。

为单个函数处理多个http方法和路由
您可以按端点或函数分隔spec

from flasgger.utils import swag_from

@app.route('/api/<string:username>', endpoint='with_user_name', methods=['PUT', 'GET'])
@app.route('/api/', endpoint='without_user_name')
@swag_from('path/to/external_file.yml', endpoint='with_user_name')
@swag_from('path/to/external_file_no_user_get.yml', endpoint='without_user_name', methods=['GET'])
@swag_from('path/to/external_file_no_user_put.yml', endpoint='without_user_name', methods=['PUT'])
def fromfile_decorated(username=None):
    if not username:
        return "No user!"
    return jsonify({'username': username})
同样，可以通过多次注册url_rule实现在同一个MethodView或SwaggerView中使用多个函数。examples/example_app

使用相同的数据来验证API POST主体(body)。
将swag_from的_validation_参数设置为True会自动验证传入的数据：

from flasgger import swag_from

@swag_from('defs.yml', validation=True)
def post():
    # 如果未通过验证，则返回状态为400的ValidationError响应并还返回验证消息。
也可以使用swagger.validate注释：

from flasgger import Swagger

swagger = Swagger(app)

@swagger.validate('UserSchema')
def post():
    '''
    file: defs.yml
    '''
    # 如果未通过验证，则返回状态为400的ValidationError响应并还返回验证消息。
Yet you can call validate manually:

from flasgger import swag_from, validate

@swag_from('defs.yml')
def post():
    validate(request.json, 'UserSchema', 'defs.yml')
    # 如果未通过验证，则返回状态为400的ValidationError响应并还返回验证消息。
也可以在SwaggerView中定义validation=True并使用 specs_dict用于验证。

请查看examples/validation.py了解更多信息。

所有验证选项均可在 http://json-schema.org/latest/json-schema-validation.html 找到

自定义验证
默认情况下，Flasgger将使用python-jsonschema 执行验证。

只要满足要求，就支持自定义验证功能：

仅接受两个位置参数：
首先要验证的数据；和
作为第二个参数验证的schema
验证失败时引发任何形式的异常。
任何返回值都将被丢弃。

向Swagger实例提供函数将使其成为默认值：

from flasgger import Swagger

swagger = Swagger(app, validation_function=my_validation_function)
提供函数作为swag_from或swagger.validate的参数 批注或直接添加到validate函数将强制使用它 而不是Swagger的默认验证功能：

from flasgger import swag_from

@swag_from('spec.yml', validation=True, validation_function=my_function)
...
from flasgger import Swagger

swagger = Swagger(app)

@swagger.validate('Pet', validation_function=my_function)
...
from flasgger import validate

...

    validate(
        request.json, 'Pet', 'defs.yml', validation_function=my_function)
验证错误处理
默认情况下，Flasgger处理验证错误的方式是中止请求(abort)并返回 带有错误消息的400 BAD REQUEST响应。

可以提供自定义验证错误处理函数(custom validation error handling function) 只要符合要求，就会取代默认行为：

接受三个且仅三个位置参数：
抛出的error
造成验证失败的数据；和
用于验证的schema
向Swagger实例提供函数将使其成为默认值：

from flasgger import Swagger

swagger = Swagger(app, validation_error_handler=my_handler)
提供这个函数作为swag_from或swagger.validate批注的参数 或直接添加到validate函数都将强制使用它 而不是Swagger的默认验证功能：

from flasgger import swag_from

@swag_from(
    'spec.yml', validation=True, validation_error_handler=my_handler)
...
from flasgger import Swagger

swagger = Swagger(app)

@swagger.validate('Pet', validation_error_handler=my_handler)
...
from flasgger import validate

...

    validate(
        request.json, 'Pet', 'defs.yml',
        validation_error_handler=my_handler)
使用自定义验证错误处理函数的应用示例： 见 example validation_error_handler.py

获取已定义的schema作为python词
您可能希望不复制spec而将在Swagger spec中定义schemas用作字典。 为此，您可以使用get_schema 函数：

from flask import Flask, jsonify
from flasgger import Swagger, swag_from

app = Flask(__name__)
swagger = Swagger(app)

@swagger.validate('Product')
def post():
    """
    post endpoint
    ---
    tags:
      - products
    parameters:
      - name: body
        in: body
        required: true
        schema:
          id: Product
          required:
            - name
          properties:
            name:
              type: string
              description: The product's name.
              default: "Guarana"
    responses:
      200:
        description: The product inserted in the database
        schema:
          $ref: '#/definitions/Product'
    """
    rv = db.insert(request.json)
    return jsonify(rv)

...

product_schema = swagger.get_schema('product')
此方法返回包含Flasgger schema id的字典， 所有定义的参数和所需参数的列表。

HTML清理器(HTML sanitizer)
默认情况下，Flasgger将尝试清理YAML定义中的内容 用<br>替换每个\n，但是您可以更改此行为 设置另一种清理器。

from flasgger import Swagger, NO_SANITIZER

app =Flask()
swagger = Swagger(app, sanitizer=NO_SANITIZER)
您可以自行编写清理器

swagger = Swagger(app, sanitizer=lambda text: do_anything_with(text))
如果您希望能够渲染您的specs description中的Markdown，还可以使用Markdown解析器MK_SANITIZER

Swagger UI和模板
您可以在应用程序中重写(override) templates/flasgger/index.html， 重写后的模板将为SwaggerUI的index.html。使用flasgger/ui2/templates/index.html 作为您自定义index.html文件的基础。

Flasgger支持Swagger UI版本2和3。版本3仍处于试验阶段，但是您 可以尝试设置app.config['SWAGGER']['uiversion'].

app = Flask(__name__)
app.config['SWAGGER'] = {
    'title': 'My API',
    'uiversion': 3
}
swagger = Swagger(app)
OpenAPI 3.0支持
对OpenAPI 3.0的实验性支持应该在使用SwaggerUI 3时起作用。要使用OpenAPI 3.0，请将app.config['SWAGGER']['openapi'] 设置为当前SwaggerUI 3支持的版本，例如'3.0.2'。

有关使用callbacks和requestBody的示例，请参见callbacks example。

从外部加载Swagger UI和jQuery JS/CSS
从Flasgger 0.9.2开始，您可以指定外部URL位置，以为Flasgger默认模板中加载的Swagger和jQuery库加载JavaScript和CSS。如果省略以下配置属性，则Flasgger将提供它所包含的静态版本-这些版本可能比当前的Swagger UI v2或v3版本要旧。

以下示例从unpkg.com加载Swagger UI和jQuery版本：

swagger_config = Swagger.DEFAULT_CONFIG
swagger_config['swagger_ui_bundle_js'] = '//unpkg.com/swagger-ui-dist@3/swagger-ui-bundle.js'
swagger_config['swagger_ui_standalone_preset_js'] = '//unpkg.com/swagger-ui-dist@3/swagger-ui-standalone-preset.js'
swagger_config['jquery_js'] = '//unpkg.com/jquery@2.2.4/dist/jquery.min.js'
swagger_config['swagger_ui_css'] = '//unpkg.com/swagger-ui-dist@3/swagger-ui.css'
Swagger(app, config=swagger_config)
Initializing Flasgger with default data.
您可以使用默认数据来启动Swagger spec或提供模板：

template = {
  "swagger": "2.0",
  "info": {
    "title": "My API",
    "description": "API for my data",
    "contact": {
      "responsibleOrganization": "ME",
      "responsibleDeveloper": "Me",
      "email": "me@me.com",
      "url": "www.me.com",
    },
    "termsOfService": "http://me.com/terms",
    "version": "0.0.1"
  },
  "host": "mysite.com",  # overrides localhost:500
  "basePath": "/api",  # base bash for blueprint registration
  "schemes": [
    "http",
    "https"
  ],
  "operationId": "getmyData"
}

swagger = Swagger(app, template=template)
And then the template is the default data unless some view changes it. You can also provide all your specs as template and have no views. Or views in external APP.

然后，除非某些视图更改了模板，否则模板是默认数据。您 也可以提供所有spec作为模板，而无需包括视图。或 外部APP中的视图。

#在运行时获取默认数据
有时您需要在运行时获取一些数据，具体取决于动态值，例如：要检查request.is_secure以确定schemes是否为https，您可以使用LazyString做到这一点。

from flask import Flask
from flasgger import, Swagger, LazyString, LazyJSONEncoder

app = Flask(__init__)

# 设置自定义编码器（如果需要自定义，则继承它）
app.json_encoder = LazyJSONEncoder


template = dict(
    info={
        'title': LazyString(lambda: 'Lazy Title'),
        'version': LazyString(lambda: '99.9.9'),
        'description': LazyString(lambda: 'Hello Lazy World'),
        'termsOfService': LazyString(lambda: '/there_is_no_tos')
    },
    host=LazyString(lambda: request.host),
    schemes=[LazyString(lambda: 'https' if request.is_secure else 'http')],
    foo=LazyString(lambda: "Bar")
)
Swagger(app, template=template)
The LazyString values will be evaluated only when jsonify encodes the value at runtime, so you have access to Flask request, session, g, etc.. and also may want to access a database. 仅当jsonify在运行时对该值进行编码时，才会评估LazyString的值，因此您可以访问Flask的request，session，g 等，或者是访问数据库。

经过反向代理
有时，您是在反向代理（例如NGINX）后面提供庞大的文档。当遵循Flask指南时， Swagger的文档将正确加载，但是“尝试一下”按钮指向错误的位置。可以使用以下代码解决此问题：

from flask import Flask, request
from flasgger import Swagger, LazyString, LazyJSONEncoder

app = Flask(__name__)
app.json_encoder = LazyJSONEncoder

template = dict(swaggerUiPrefix=LazyString(lambda : request.environ.get('HTTP_X_SCRIPT_NAME', '')))
swagger = Swagger(app, template=template)
自定义默认配置
可以向Flasgger提供自定义配置，例如不同的spec route或禁用Swagger UI：

swagger_config = {
    "headers": [
    ],
    "specs": [
        {
            "endpoint": 'apispec_1',
            "route": '/apispec_1.json',
            "rule_filter": lambda rule: True,  # all in
            "model_filter": lambda tag: True,  # all in
        }
    ],
    "static_url_path": "/flasgger_static",
    # "static_folder": "static",  # must be set by user
    "swagger_ui": True,
    "specs_route": "/apidocs/"
}

swagger = Swagger(app, config=swagger_config)
提取定义
当在spec中找到id时，可以提取定义，例如：

from flask import Flask, jsonify
from flasgger import Swagger

app = Flask(__name__)
swagger = Swagger(app)

@app.route('/colors/<palette>/')
def colors(palette):
    """Example endpoint returning a list of colors by palette
    ---
    parameters:
      - name: palette
        in: path
        type: string
        enum: ['all', 'rgb', 'cmyk']
        required: true
        default: all
    responses:
      200:
        description: 颜色列表（可被调色板过滤）
        schema:
          id: Palette
          type: object
          properties:
            palette_name:
              type: array
              items:
                schema:
                  id: Color
                  type: string
        examples:
          rgb: ['red', 'green', 'blue']
    """
    all_colors = {
        'cmyk': ['cyan', 'magenta', 'yellow', 'black'],
        'rgb': ['red', 'green', 'blue']
    }
    if palette == 'all':
        result = all_colors
    else:
        result = {palette: all_colors.get(palette)}

    return jsonify(result)

app.run(debug=True)
在此示例中，您不必传递definitions，而是需要将id添加到您的schema。

Python2兼容性
版本0.9.5.*将是最后一个支持Python2的版本。 请在这里讨论#399。
