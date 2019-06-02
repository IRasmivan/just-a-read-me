## 1. Description:
This project is an Rest API, developed in JAVA using spring framework. It have the below functionality implemented on the below area.

The one listed below as :pushpin: are the list of are part of the excisies.

### 1.1 Stock
* Get the stock for provided productId :pushpin:
* Add stock for an product
* Updated stock for an existing stock :pushpin:

### 1.2 Statistic
* Get the statistic for today or lastMonth, listing TopAvailableProduct and TopSellingProduct for the given range. :pushpin:

### 1.3 Product
* Adding of product
* Get all the products from backend
* Get the product and the associated stock for a given productId

### 1.4 Invoice
* Add invoice for an product to record sales.

---

## 2. Tools & framework:
The below are the list of tools and framework used in the project!
* Spring Boot framework
* Maven for Packaging and Build
* Java Programming language
* MySQL as backend

---

## 3. Stock Endpoint:

The below are the endpoints that are exposed to GET PUT PATCH Stock!

### 3.1 GET Stock By Product Id

This endpoint will list all the stock that are available for the given ProductId. This endpoint covers the below scenarios.

#### Description of scenario covered:
* To enable **CONCURRENT REQUEST** for getting the stock for the productId. If stock is found for the productId, then populate response header with key **ETAG** and value for the key with **VERSION** column value from stock table for the given productId.
* The requested stock for the given productid will be searched from the inmemory **CACHE**, if available the data will be served to the user else it will be searched from the stock table.
* If there are **NO STOCK AVAILABLE** for the productId, then I have populated an attribute called StockMessage which has message as 'There are no stock available for this productId'.
* Check if the user have given valid productId for the stock. If it is **INVALID PRODUCTID**, then respond user as ErroneousJsonException with message as 'Product not found'.

### 3.1.1 Getting stock for valid productid

#### Request - Valid ProductId
```
GET /commercetools/api/v1/stock?productId=veg HTTP/1.1
Host: localhost:8080
Content-Type: application/json
Authorization: Basic dXNlcjpwYXNzd29yZA==
```

#### Response - Valid ProductId

##### Response Body
```
{
    "productId": "veg",
    "requestTimestamp": "2019-06-01T19:00:54.248Z",
    "stock": [
        {
            "id": "1",
            "timestamp": "2019-05-31T09:53:53.390Z",
            "quantity": 229
        }
    ]
}
```

##### Response Header
```
ETag: "20"
```

---

### 3.1.2 Getting stock for invalid productid
#### Request - InValid ProductId
```
GET /commercetools/api/v1/stock?productId=cake HTTP/1.1
Host: localhost:8080
Content-Type: application/json
Authorization: Basic dXNlcjpwYXNzd29yZA==
```

#### Response - InValid ProductId
```
{
    "message": "Invalid Request :: Reason - Product not found",
    "statusCode": "BAD_REQUEST"
}
```

---

### 3.2 Update existing stock
This endpoint will help in updating the existing stock.

#### Description of scenario covered:
* If user provide a invalid productId in the request JSON, then respond user as ErroneousJsonException with a message 'Invalid Request :: Reason - Product not found'.
* If user provide a invalid stockId in the request JSON, then respond user as ErroneousJsonException with a message 'Invalid Request :: Reason - Invalid stockid'.
* If user provide a timestamp for an existing stock with an earlier timestamp to update, then respond with HTTP status as 204 No Content.
* If a valid JSON request is provided. Since I have implemented cache, the cached values are refreshed so that the GET request can use the data from cache.

##### Special Case
For concurrent requests for updating the same stock. 
* If a Stock is found for a productId, Then
	+ The request HTTP header is checked for a key 'if-Match' and the value from if-Match key is validated with the current stock version. if TRUE,  then
		+ The stock is updated
	+ else, http status precondition failed exception is thrown to user. 

### 3.2.1 Update stock for Invalid JSON (One Sample Request)

#### Request
```
POST /commercetools/api/v1/stock/update HTTP/1.1
Host: localhost:8080
Content-Type: application/json
If-Match: 20
Authorization: Basic dXNlcjpwYXNzd29yZA==
{
	"id":"100",
	"productId": "veg",
	"timestamp": "2019-05-30T09:53:53.390Z",
	"quantity": 229
}
```

#### Response
```
{
    "message": "Invalid Request :: Reason - Invalid stockid",
    "statusCode": "BAD_REQUEST"
}
```

---

### 3.2.2 Update stock for Valid JSON (One Sample Request)

#### Request
```
POST /commercetools/api/v1/stock/update HTTP/1.1
Host: localhost:8080
Content-Type: application/json
If-Match: 20
Authorization: Basic dXNlcjpwYXNzd29yZA==
{
	"id":"1",
	"productId": "veg",
	"timestamp": "2019-05-31T10:53:53.390Z",
	"quantity": 229
}
```


#### Response
```
{
    "productId": "veg",
    "requestTimestamp": "2019-06-01T21:31:25.167Z",
    "stock": [
        {
            "id": "1",
            "timestamp": "2019-05-31T10:53:53.390Z",
            "quantity": 229
        }
    ]
}
```
---
### 3.3 Add new stock: __(Additional Feature)__

This endpoint is only to add a new stock. I have not added any validation for this endpoint in particular.

#### Request
```
PUT /commercetools/api/v1/stock/add HTTP/1.1
Host: localhost:8080
Content-Type: application/json
Authorization: Basic dXNlcjpwYXNzd29yZA==
{
	"productId": "drinks",
	"timestamp": "2019-05-29T22:55:01.754Z",
	"quantity": 98
}
```
#### Response
````
{
    "productId": "drinks",
    "requestTimestamp": "2019-05-31T18:43:41.776Z",
    "stock": [
        {
            "id": "2",
            "timestamp": "2019-05-29T22:55:01.754Z",
            "quantity": 98
        }
    ]
}
````
---
## 4. Statistic Endpoint
This endpoint list the top 3 available stock and top 3 sales products for the users.

### 4.1 Getting Statistic

#### Description of scenario:
* Check if the User has given a valid time (today or lastMonth). If its invalid time, then respond user as InvalidStatisticTimeException with message as 'Invalid time for statistics, Please provide a valid value'.
* If there are no stock available for the given time, then I have populated an attribute called TopAvailableProductMessage which has message as "There are no product that was available for provided 'time'".
* If there are stock available but less than 3 for the given time, then I have populated an attribute called TopAvailableProductMessage which has message as "There are only 'count of available stock' product that had sales for 'time'".
* If there are no sales available for the given time, then I have populated an attribute called TopSellingProductsMessage which has message as 'There are no product that was available for provided 'time'".
* If there are stock sales but less than 3 for the given time, then I have populated an attribute called TopSellingProductsMessage which has message as 'There are only 'count of sales' product that had sales for 'time'".


#### Request
```
GET /commercetools/api/v1/statistics?time=lastMonth HTTP/1.1
Host: localhost:8080
Authorization: Basic dXNlcjpwYXNzd29yZA==
```

#### Response
```
{
    "requestTimestamp": "2019-06-01T21:44:56.073Z",
    "range": "lastMonth",
    "topAvailableProductMessage": "There are only 2 product that had available for lastMonth",
    "topAvailableProducts": [
        {
            "id": "1",
            "timestamp": "2019-05-31T10:53:53.390Z",
            "productId": "veg",
            "quantity": 229
        },
        {
            "id": "2",
            "timestamp": "2019-05-29T22:55:01.754Z",
            "productId": "drinks",
            "quantity": 98
        }
    ],
    "topSellingProductsMessage": "There are no sales available on any product for lastMonth"
}
```
---

## 5. Product Endpoint: __(Additional Feature)__

The below are the endpoints that are exposed to GET PUT products!

### 5.1 GET Product
This endpoint gets all the products from database.

#### Request
```
GET /commercetools/api/v1/products HTTP/1.1
Host: localhost:8080
Content-Type: application/json
Authorization: Basic dXNlcjpwYXNzd29yZA==
```
#### Response
```
[
    {
        "id": 1,
        "productId": "cake",
        "stock": [
            {
                "id": "2",
                "quantity": 100,
                "timestamp": "2019-05-29T22:55:01.754Z"
            }
        ],
        "invoice": [
            {
                "id": "1",
                "quantity": 99,
                "timestamp": "2019-05-29T22:55:01.754Z"
            }
        ]
    }
]
```

---

### 5.2 PUT Product
This endpoint allows the user to add a new product.
#### Request
```
PUT /commercetools/api/v1/product/add HTTP/1.1
Host: localhost:8080
Content-Type: application/json
Authorization: Basic dXNlcjpwYXNzd29yZA==
{
	"productId": "drinks"
}
```

#### Response
```
<<Auto Generated id>>
```
---
### 5.3 GET Product by productId

This endpoint gets the product for a given productId from database.

#### Request
```
GET /commercetools/api/v1/product/stock?productId=drinks HTTP/1.1
Host: localhost:8080
Content-Type: application/json
Authorization: Basic dXNlcjpwYXNzd29yZA==
```
#### Response
```
{
    "productId": "drinks",
    "requestTimestamp": "2019-05-31T12:09:27.652Z",
    "stock": []
}
```
---

## 6. Invoice Endpoint

To add sales for a product, I have added this endpoint so that we can add sales!

### 6.1 Add Invoice:

#### Description of scenario covered:
* Check if the productId exist for the request invoice that needs to be added, if invalid productId then respond user with message "Error while saving data  - Product not found".

##### Request

```
PUT /commercetools/api/v1/invoice/add HTTP/1.1
Host: localhost:8080
Content-Type: application/json
Authorization: Basic dXNlcjpwYXNzd29yZA==
{
	"productId": "veg",
	"timestamp": "2019-05-29T22:55:01.754Z",
	"quantity": 99
}
```

##### Response

```
{
    "id": "1",
    "quantity": 99,
    "timestamp": "2019-05-29T22:55:01.754Z"
}
```

## Extract Feature enabled:
The below are the add-on as feature.
* Swagger-UI
* Docker


### Run as Docker Container ![Docker container](https://img.icons8.com/color/50/000000/docker.png "Docker Container")

I have built a docker image and the same is available in [dockerhub](https://cloud.docker.com/u/rasmivan/repository/docker/rasmivan/commercetools). Run the below comment to run the docker image as container.
    
    docker run -p 8080:8080 -t rasmivan/commercetools:1.0

