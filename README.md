### __Getting things up and running__

Lets create a new conda env for Fast API and documentation related work. Install `fastapi` and `uvicorn`. Create two key files `main.py` and `description.py`.

```bash
conda create --name doc_engine python=3.8 -y
conda activate doc_engine
touch main.py
touch description.py
pip install "fastapi[all]"
pip install "uvicorn[standard]"
```

```python
# main.py
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
async def root():
    return {"message": "Hello World"}
```

Save the above content in `main.py`. 

```bash
uvicorn main:app --reload
```
Execute the above lines to check the Fast API server is running or not. Check `http://127.0.0.1:8000` and `http://127.0.0.1:8000/docs`. If the landing page and Swagger UI page is coming up, we are good to go.

### __API level documentation__

Here in API level documentation we are going capture the what this API collection is about. What are the overall endpoints present in this collection and the high level objective of these endpoints. Also we can keep track of a what all methods we have implemented and what other we need to develop.

We will update the following code in `description.py`. As it is visible here that it is used to describe the high level objectives `Items` and `Users`.

```python
# description.py
description = """
ChimichangApp API helps you do awesome stuff. 🚀

## Items

You can **read items**.

## Users

You will be able to:

* **Create users** (_not implemented_).
* **Read users** (_not implemented_).
"""
```

Along with the description we can use the following information regarding the API collection in `main.py`.

```python
# main.py
from description import description

app = FastAPI(
    title="ChimichangApp",
    description=description,
    version="0.0.1",
    terms_of_service="http://example.com/terms/",
    contact={
        "name": "Deadpoolio the Amazing",
        "url": "http://x-force.example.com/contact/",
        "email": "dp@x-force.example.com",
    },
    license_info={
        "name": "Apache 2.0",
        "url": "https://www.apache.org/licenses/LICENSE-2.0.html",
    },
)

@app.get("/items/")
async def read_items():
    return [{"name": "Katana"}]
```

### __Tag level documentation__

Here we can create sub-collection of APIs which are known as `tags`. This can be a group of APIs doing operation related to a business logic or objective. Some example of these can be `user-management`, `project-management`, `data-management`. These are quite similar to the routes we have in Fast API.

```python
# description.py
tags_metadata = [
    {
        "name": "users",
        "description": "Operations with users. The **login** logic is also here.",
    },
    {
        "name": "items",
        "description": "Manage items. So _fancy_ they have their own docs.",
        "externalDocs": {
            "description": "Items external docs",
            "url": "https://fastapi.tiangolo.com/",
        },
    },
]
```

```python
# main.py
from description import description, tags_metadata

app = FastAPI(
    title="ChimichangApp",
    description=description,
    version="0.0.1",
    terms_of_service="http://example.com/terms/",
    contact={
        "name": "Deadpoolio the Amazing",
        "url": "http://x-force.example.com/contact/",
        "email": "dp@x-force.example.com",
    },
    license_info={
        "name": "Apache 2.0",
        "url": "https://www.apache.org/licenses/LICENSE-2.0.html",
    },
    openapi_tags=tags_metadata)

@app.get("/users/", tags=["users"])
async def get_users():
    return [{"name": "Harry"}, {"name": "Ron"}]


@app.get("/items/", tags=["items"])
async def get_items():
    return [{"name": "wand"}, {"name": "flying broom"}]
```

### __Schema or Data Model level documentation__

Here is an example how we can create a DB model, request and response level documentation. There are other ways to do the same thing we have done here using `Field`. Check the reference section for including multiple schema example.

```python
# main.py

from pydantic import BaseModel, Field
from typing import Union

class Item(BaseModel):
    name: str = Field(example="Foo")
    description: Union[str, None] = Field(default=None, example="A very nice Item")
    price: float = Field(example=35.4)
    tax: Union[float, None] = Field(default=None, example=3.2)

@app.put("/items/{item_id}", tags=["items"])
async def update_item(item_id: int, item: Item):
    results = {"item_id": item_id, "item": item}
    return results
```


### __Endpoint level documentation__

If we want to add details regarding a endpoint we can use `description` arg in any method to do that. We can capture the inner working of a endpoint using this argument. Also, note that we can use `title` and `description` for a `query_parameter` in `GET` method to understand how it is being used in the endpoint.

```python
# main.py

from fastapi import FastAPI, Query

@app.get("/new_items/", tags=["items"], description = "This API is for creating new items.")
async def read_items(
    q: Union[str, None] = Query(
        default=None,
        title="Query string",
        description="Query string for the items to search in the database that have a good match",
        min_length=3,
    )
):
    results = {"items": [{"item_id": "Foo"}, {"item_id": "Bar"}]}
    if q:
        results.update({"q": q})
    return results
```

### __Response documentation__

We can use `response` argument to pass status code along with the description about the response. Check [this](https://betterprogramming.pub/metadata-and-additional-responses-in-fastapi-ea90a321d477) blog post for details.

```python
users = {"000": "admin","001": "Wai Foong", "002": "Jane", "003": "Jessie", "007": "Five Six Seven"}

@app.get("/get-user", tags=["get-user"], response_model=Result,
    responses={
        403: {
            "model": Message, 
            "description": "Insufficient privileges for this action"
        },
        404: {
            "model": Message,
            "description": "No user with this ID in the database",
        },
        200: {
            "description": "Successfully retrieved information of the user",
            "content": {
                "application/json": {
                    "example":  {'status_code': '0', 'status_message' : 'Success', 'data': {'id': '001', 'name': 'John Doe'}}
                }
            },
        },
    },)
async def get_user(id: str = Query(..., title="3-digit identity number of the user", example="010")):
    if id in users:
        if id == "007":
            return JSONResponse(status_code=403, content={"message": "Insufficient privileges!"})

        return {"status_code": "0", "status_message" : "Success", "data": {"id": id, "name": users[id]}}
    else:
        return JSONResponse(status_code=404, content={"message": "User not found!"})
```


```bash
conda deactivate
```