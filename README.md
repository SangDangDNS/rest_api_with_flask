# rest_api_with_flask

Firstly, we create a virtual environment with purpose to save every install in this virtual environment. Enter this command: `python -m venv .env` and then we will active it by command `source .env/bin/activate`.  
The result like this: 
```
(base) sang@sang-desktop:~/Desktop/rest_api_with_flask$ source .env/bin/activate
(.env) (base) sang@sang-desktop:~/Desktop/rest_api_with_flask$ 
```
Next, we will install `flask` by command: `pip install flask` and to inspect whether flask is installed by command: `pip list`. This is the result: 
```
(.env) (base) sang@sang-desktop:~/Desktop/rest_api_with_flask$ pip list
Package      Version
------------ -------
blinker      1.6.3
click        8.1.7
Flask        3.0.0
itsdangerous 2.1.2
Jinja2       3.1.2
MarkupSafe   2.1.3
pip          23.2.1
setuptools   65.5.0
Werkzeug     3.0.0

[notice] A new release of pip is available: 23.2.1 -> 23.3
[notice] To update, run: pip install --upgrade pip
(.env) (base) sang@sang-desktop:~/Desktop/rest_api_with_flask$
```
Create a file `app.py` with this code: 
```
from flask import Flask

app = Flask(__name__)
```
We run it in terminal by command: `flask run`

And now, we will add a list of store in this program and then create the endpoint to get that data from store.  
File `app.py`:
```
from flask import Flask

app = Flask(__name__)

stores = [
    {
        "name": "My store",
        "items": [
            {
                     "name": "Chair",
            "price": 15.99
            }       
        ]
    }
]

@app.get("/store")
def get_stores():
    return {"stores": stores}
```
So, run it in terminal and access this url to view result: `http://127.0.0.1:5000/store`

![Alt text](imgs/image-1.png)

#### Interact and Test REST API:
We will use Postman desktop to test API:
![Alt text](imgs/image-2.png)

Next, in Postman we add a method `POST`:
![Alt text](imgs/image-3.png)  
Then we will create store by endpoint in program and add a object of flask is `request` like this:
File `app.py`:
```
from flask import Flask, request

app = Flask(__name__)

stores = [
    {
        "name": "My store",
        "items": [
            {
                     "name": "Chair",
            "price": 15.99
            }       
        ]
    }
]

@app.get("/store")
def get_stores():
    return {"stores": stores}


@app.post("/store")
def post_stores():
    request_data = request.get_json()
    new_store = {"name": request_data["name"], "items": []}
    stores.append(new_store)
    return stores, 201
```
`201` is status of method POST, it mean create successfully.  
In postman, click `Send`, then view result:
![Alt text](imgs/image-4.png)
![Alt text](imgs/image-5.png)
![Alt text](imgs/image.png)

But if you run again program, it will only appear the first store, so the best way is we store in database.
![Alt text](imgs/image-6.png)

Next, we will create item in store.
File `app.py`:
```
from flask import Flask, request

app = Flask(__name__)

stores = [
    {
        "name": "My store",
        "items": [
            {
                "name": "Chair",
                "price": 15.99
            }       
        ]
    }
]

@app.get("/store")
def get_stores():
    return {"stores": stores}


@app.post("/store")
def post_stores():
    request_data = request.get_json()
    new_store = {"name": request_data["name"], "items": []}
    stores.append(new_store)
    return stores, 201

@app.post("/store/<string:name>/item")
def create_name(name):
    request_data = request.get_json()
    for store in stores:
        if store["name"] == name:
            new_item = {"name": request_data["name"], "price": request_data["price"]}
            store["item"].append(new_item)
            return stores, 201
    return {"message": "Store not found"}, 404
```
In Postman, we create new method `POST` to send data.
![Alt text](imgs/image-7.png)
Click send and then choose method `GET` to view result:
![Alt text](imgs/image-8.png)
![Alt text](imgs/image-9.png)
View in Browser:
![Alt text](imgs/image-10.png)
So, now we will add item for `My store 2`, but firstly we have to create store 2nd.
![Alt text](imgs/image-11.png)
Next, we adjust name in method post of creating item.
![Alt text](imgs/image-12.png)
View result:
![Alt text](imgs/image-13.png)
and on Browser:
![Alt text](imgs/image-14.png)

#### Get specific store and its items:

In `app.py` we add 2 methods to get specific store and its items:
```
@app.get("/store/<string:name>")
def get_store(name):
    for store in stores:
        if store["name"] ==  name:
            return store
    return {"message": "Store not found"}, 404


@app.get("/store/<string:name>/item")
def get_item_in_store(name):
    for store in stores:
        if store["name"] == name:
            return {"items": store["items"]}
    return {"message": "Store not found"}, 404
```

And in Postman, we also create 2 method GET.
![Alt text](imgs/image-15.png)
![Alt text](imgs/image-16.png)
Because the 2nd store not yet created. So we will receive the message like this:
![Alt text](imgs/image-17.png)

#### Docker:
We will create a Dockerfile to build the image Docker like this
File `Dockerfile`:
```
FROM python:3.11
EXPOSE 5000
WORKDIR /app
RUN pip install flask
COPY . .
CMD [ "flask", "run", "--host", "0.0.0.0" ]
```
Next, we will build this Dockerfile by command: `sudo docker build -t rest-api-flask .`  
The result like this:

```
(.env) (base) sang@sang-desktop:~/Desktop/rest_api_with_flask$ sudo docker build -t rest-api-flask .
[+] Building 1.0s (9/9) FINISHED                                                                                                         docker:default
 => [internal] load .dockerignore                                                                                                                  0.0s
 => => transferring context: 2B                                                                                                                    0.0s
 => [internal] load build definition from Dockerfile                                                                                               0.0s
 => => transferring dockerfile: 153B                                                                                                               0.0s
 => [internal] load metadata for docker.io/library/python:3.11                                                                                     0.8s
 => [1/4] FROM docker.io/library/python:3.11@sha256:652c9f890a7f38bab4d67ee95c54d72955792623122cfea0a87aa74d927e41ae                               0.0s
 => [internal] load build context                                                                                                                  0.0s
 => => transferring context: 197.72kB                                                                                                              0.0s
 => CACHED [2/4] WORKDIR /app                                                                                                                      0.0s
 => CACHED [3/4] RUN pip install flask                                                                                                             0.0s
 => CACHED [4/4] COPY . .                                                                                                                          0.0s
 => exporting to image                                                                                                                             0.0s
 => => exporting layers                                                                                                                            0.0s
 => => writing image sha256:7055adce0b0d681d6825986d1beeedd9aecb03f04049baf7cc43cf633cc5129b                                                       0.0s
 => => naming to docker.io/library/rest-api-flask                        
```

We will run that image by command: `sudo docker run -d -p 5005:5000 rest-api-flask`
Result:
```
(.env) (base) sang@sang-desktop:~/Desktop/rest_api_with_flask$ sudo docker run -d -p 5005:5000 rest-api-flask
5858edfc7ff66c8a5043f2d180311cf930320bbf25c1014b3eb132b1f24560b5
```
And we will back to Postman and get data:
![Alt text](imgs/image-18.png)

Why we receive the error as above. Due to in Dockerfile, we use expose port is 5000 and when we run image, we assign
port 5005 to refer to port of container. Thus, in Postman, if we use port 5000 we will get an error, we have to use
port 5005.
![Alt text](imgs/image-19.png)

#### Postgresql:
So now, we will mention to database to store data. We will use Postgresql in Docker.
Firsly, we need to pull Postgres for Docker by command `docker pull postgres`. Check docker images 
by command `sudo docker images`.  
Next, we create a docker-compose file to run container postgres. But in this file, we will have some sensitive datas.
So in the same directory, we add a file `.env` to store those sensitive datas. And this file will not public.  
File `.env`:
```text
POSTGRES_USER=sang
POSTGRES_PASSWORD=sang1234
POSTGRES_DB=store
url="postgresql://sang:sang1234@localhost:5432/store"
```
Now, we will add a file `docker-compose.yaml`  
File `docker-compose.yaml`
```text
services:
  pgdatabase:
    image: postgres:13
    environment:
      - POSTGRES_USER=${POSTGRES_USER}
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
      - POSTGRES_DB=${POSTGRES_DB}
    volumes:
      - "./store_data:/var/lib/postgresql/data:rw"
    ports:
      - 5432:5432
```
When we run that docker-compose file, it will get value of variable environment in file `.env`.  
Run `sudo docker-compose up -d`  
The result:  
```text
(.myenv) (base) sang@sang-desktop:~/Desktop/rest_api_with_flask$ sudo docker-compose up -d
[sudo] password for sang: 
[+] Running 2/2
 ✔ Network rest_api_with_flask_default         Created                                                                                                       0.1s 
 ✔ Container rest_api_with_flask-pgdatabase-1  Started                                                                                                       0.3s 
(.myenv) (base) sang@sang-desktop:~/Desktop/rest_api_with_flask$ sudo docker ps
CONTAINER ID   IMAGE         COMMAND                  CREATED          STATUS          PORTS                                       NAMES
defb60d43ed5   postgres:13   "docker-entrypoint.s…"   44 seconds ago   Up 43 seconds   0.0.0.0:5432->5432/tcp, :::5432->5432/tcp   rest_api_with_flask-pgdatabase-1
```  
To access database we can also use command: `psql -U sang -h localhost -d store -p 5432` and enter password when have requirement.

#### Run API in Docker with automatic reloading and debug mode
Firstly, we will add a new file `requirements.txt` to store modules we need to install.
File `requirements.txt`:
```
flask
flask-smorest
python-dotenv
```
Next we will add another file `.flaskenv`
File `.flaskenv`:
```
FLASK_APP=app
FLASK_ENV=development
```
The main purpose of that file is running automatically the flask app when we change something, that we do not have to
rerun manually.  
Right now, we will update the `Dockerfile`
File `Dockerfile`:
```text
FROM python:3.11
EXPOSE 5000
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
CMD [ "flask", "run", "--host", "0.0.0.0" ]
```
Then we build image again: `sudo docker build -t flask-smorest-api .`
And run it: `sudo docker run -p 5000:5000 flask-rest-api`
But, we have a problem when we follow that way. Every time we change code in file `app.py` so we have to rebuild and rerun.  
We will run this command: `sudo docker run -p 5000:5000 -w /app -v "$(pwd):/app" flask-smore-api`
This is workflow: when we change code in `app.py`, the Flask app will refresh because we have file `.flaskenv`, and befofe
we have run docker use refer like this `-v "$(pwd):/app`. Thus, we do not need to rerun or rebuild docker.
![img.png](imgs%2Fimg.png)
Note: that way should use for the case you deploy your app on local.
Okay, now we will combine Dockerfile and Docker-compose. In any project, we will create Dockerfile firstly, then we
will add a Docker-compose file.  
File `Dockerfile`:
```text
FROM python:3.11
EXPOSE 5000
WORKDIR /app
COPY requirements.txt .
RUN pip install --upgrade Flask
RUN pip install --upgrade Flask-Smorest
RUN pip install -r requirements.txt
COPY . .
CMD ["flask", "run", "--host", "0.0.0.0"]
```
File `docker-compose.yaml`:
```text
services:
  pgdatabase:
    image: postgres:13
    environment:
      - POSTGRES_USER=${POSTGRES_USER}
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
      - POSTGRES_DB=${POSTGRES_DB}
    volumes:
      - "./store_data:/var/lib/postgresql/data:rw"
    ports:
      - 5432:5432

  flaskapp:
    volumes:
    - ".:/app"
    build: .
    ports:
      - 5000:5000
    depends_on:
      - pgdatabase
```
We just run the docker-compose file by command: `sudo docker-compose up -d`. If you wanna rebuild service, run this
command: `sudo docker-compose up --build`
#### SQLAlchemy:
Why we use SQLAlchemy, because it has some benefits:
- ORM (object relational mapping)
- Multi-Threading
- Support to create the tables and their columns

So, we have SQLAlchemy Core and SQLAlchemy ORM, in this project, We will focus on SQLAlchemy ORM.  
Why don't we access directly database but through ORM of SQLAlchemy??  
So we have some advantages when use ORM of SQLAlchemy:
1. Abtracsion DB: ORM permit us to interact with database via object python instead of write query directly sql.
It helps to easily readable and maintenance source code.
2. Portability across different database systems: we can change DBMS without change too much application code.
3. Error handling: It can provide clear and readable error information for you to understand and handle errors 
more effectively.
4. Security: Using ORM can help protect your application against SQL injection attacks by automatically 
handling query parameters.

Now, we creata file `db.py`  
File `db.py`  
```text
from flask_sqlalchemy import SQLAlchemy

db = SQLAlchemy()
```
Next, we create a folder `models`. Inside this folder, we create 3 files `item.py`, `store.py` and `__init__.py`    
File `store.py`:
```text
from db import db


class StoreModel(db.Model):
    __tablename__ = "stores"

    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(80), unique=True, nullable=False)
    items = db.relationship("ItemModel", back_populates="store", lazy="dynamic")
```
File `item.py`:
```text
from db import db


class ItemModel(db.Model):
    __tablename__ = "items"

    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(80), unique=True, nullable=False)
    price = db.Column(db.Float(precision=2), unique=False, nullable=False)
    store_id = db.Column(db.Integer, db.ForeignKey('stores.id'), unique=False, nullable=False)
    store = db.relationship("StoreModel", back_populates="items")
```
File `__init__.py`:
```text
from models.store import StoreModel
from models.item import ItemModel
```
Next, we configure Flask-SQLAlchemy with Flask app.
```text
app.config["SQLALCHEMY_DATABASE_URI"] = db_url or os.getenv("url")
app.config["SQLALCHEMY_TRACK_MODIFICATIONS"] = False
```

#### Blueprint-Methodview-Marshmallow-Decorating responses in API
We use Blueprint of Flask-Smorest, the benefit of this is help to collect method HTTP in a class.  
About Methodview, the main benefit is manage HTTP protocol, we can deal with many method like as: GET, POST, PUT, DELETE,.. 
on the same URL.    
And Marshmallow helps to validate data. That is checking the data that the client sends us, make sure it's
correct, and it adheres to the schema.   
Finally, decorating responses helps us more easily define what will
be returned for each status code.
Firstly, we create a `schemas.py` file in root directory.  
File `schamas.py`:
```text
from marshmallow import Schema, fields


class PlainItemSchema(Schema):
    id = fields.Int(dump_only=True)
    name = fields.Str(required=True)
    price = fields.Float(required=True)


class PlainStoreSchema(Schema):
    id = fields.Int(dump_only=True)
    name = fields.Str()


class ItemSchema(PlainItemSchema):
    store_id = fields.Int(required=True, load_only=True)
    store = fields.Nested(PlainStoreSchema(), dump_only=True)


class ItemUpdateSchema(Schema):
    name = fields.Str()
    price = fields.Float()


class StoreSchema(PlainStoreSchema):
    items = fields.List(fields.Nested(PlainItemSchema()), dump_only=True)
    
```
And then, we will create a folder `resources` contain 2 files `item.py` and `store.py`.  
File `item.py`:
```text
from flask.views import MethodView
from flask_smorest import Blueprint, abort
from sqlalchemy.exc import SQLAlchemyError

from db import db
from models import ItemModel
from schemas import ItemSchema, ItemUpdateSchema

blp = Blueprint("Items", "items", description="Operations on items")


@blp.route("/item/<string:item_id>")
class Item(MethodView):
    @blp.response(200, ItemSchema)
    def get(self, item_id):
        item = ItemModel.query.get_or_404(item_id)
        return item

    def delete(self, item_id):
        item = ItemModel.query.get_or_404(item_id)
        db.session.delete(item)
        db.session.commit()
        return {"message": "Item deleted."}

    @blp.arguments(ItemUpdateSchema)
    @blp.response(200, ItemSchema)
    def put(self, item_data, item_id):
        item = ItemModel.query.get(item_id)

        if item:
            item.price = item_data["price"]
            item.name = item_data["name"]
        else:
            item = ItemModel(id=item_id, **item_data)

        db.session.add(item)
        db.session.commit()

        return item


@blp.route("/item")
class ItemList(MethodView):
    @blp.response(200, ItemSchema(many=True))
    def get(self):
        return ItemModel.query.all()

    @blp.arguments(ItemSchema)
    @blp.response(201, ItemSchema)
    def post(self, item_data):
        item = ItemModel(**item_data)

        try:
            db.session.add(item)
            db.session.commit()
        except SQLAlchemyError:
            abort(500, message="An error occurred while inserting the item.")

        return item
```

File `store.py`:
```text
from flask.views import MethodView
from flask_smorest import Blueprint, abort
from sqlalchemy.exc import SQLAlchemyError, IntegrityError

from db import db
from models import StoreModel
from schemas import StoreSchema


blp = Blueprint("Stores", "stores", description="Operations on stores")


@blp.route("/store/<string:store_id>")
class Store(MethodView):
    @blp.response(200, StoreSchema)
    def get(self, store_id):
        store = StoreModel.query.get_or_404(store_id)
        return store

    def delete(self, store_id):
        store = StoreModel.query.get_or_404(store_id)
        db.session.delete(store)
        db.session.commit()
        return {"message": "Store deleted"}, 200


@blp.route("/store")
class StoreList(MethodView):
    @blp.response(200, StoreSchema(many=True))
    def get(self):
        return StoreModel.query.all()

    @blp.arguments(StoreSchema)
    @blp.response(201, StoreSchema)
    def post(self, store_data):
        store = StoreModel(**store_data)
        try:
            db.session.add(store)
            db.session.commit()
        except IntegrityError:
            abort(
                400,
                message="A store with that name already exists.",
            )
        except SQLAlchemyError:
            abort(500, message="An error occurred creating the store.")

        return store
```

Finally, we have some config in `app.py` file like this:
```text
import os
from flask import Flask
from flask_smorest import Api

from db import db
import models

from resources.item import blp as ItemBlueprint
from resources.store import blp as StoreBlueprint


def create_app(db_url=None):
    app = Flask(__name__)
    print(os.getenv("url"))

    app.config["PROPAGATE_EXCEPTIONS"] = True
    app.config["API_TITLE"] = "Stores REST API"
    app.config["API_VERSION"] = "v1"
    app.config["OPENAPI_VERSION"] = "3.0.3"
    app.config["OPENAPI_URL_PREFIX"] = "/"
    app.config["OPENAPI_SWAGGER_UI_PATH"] = "/swagger-ui"
    app.config["OPENAPI_SWAGGER_UI_URL"] = "https://cdn.jsdelivr.net/npm/swagger-ui-dist/"
    app.config["SQLALCHEMY_DATABASE_URI"] = db_url or os.getenv("url")
    app.config["SQLALCHEMY_TRACK_MODIFICATIONS"] = False
    db.init_app(app)

    api = Api(app)

    with app.app_context():
        db.create_all()

    api.register_blueprint(ItemBlueprint)
    api.register_blueprint(StoreBlueprint)

    return app

```

We will run this application by command: `sudo docker-compose up -d`
