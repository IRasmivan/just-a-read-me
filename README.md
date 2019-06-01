## Description:
This project is an Rest API, developed in JAVA using spring framework. It have the below functionality implemented on the below area.

The one listed below as :pushpin: are the list of are part of the excisies.

### Product
* Adding of product
* Get all the products from backend
* Get the product and the associated stock for a given productId

### Stock
* Get the stock for provided productId :pushpin:
* Add stock for an product
* Updated stock for an existing stock :pushpin:

### Invoice
* Add invoice for an product to record sales.

### Statistic
* Get the statistic for today or lastMonth, listing TopAvailableProduct and TopSellingProduct for the given range. :pushpin:

## Tools & framework:
The below are the list of tools and framework used in the project!
* Spring Boot framework
* Maven for Packaging and Build
* Java Programming language
* MySQL as backend

## Product Endpoint
The below are the endpoints that are exposed to GET PUT products!


### GET Product
This endpoint gets all the products from database.

```
GET /commercetools/api/v1/products HTTP/1.1
Host: localhost:8080
Content-Type: application/json
Authorization: Basic dXNlcjpwYXNzd29yZA==
```
Response from RestAPI.
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
### PUT Product
This endpoint allows the user to add a new product.

```
PUT /commercetools/api/v1/product/add HTTP/1.1
Host: localhost:8080
Content-Type: application/json
Authorization: Basic dXNlcjpwYXNzd29yZA==
{
	"productId": "drinks"
}------WebKitFormBoundary7MA4YWxkTrZu0gW--
```

Response from RestAPI
```
<<Auto Generated id>>
```


### GET Product by productId
This endpoint gets the product for a given productId from database.

```
GET /commercetools/api/v1/product/stock?productId=drinks HTTP/1.1
Host: localhost:8080
Content-Type: application/json
Authorization: Basic dXNlcjpwYXNzd29yZA==
```

Response from RestAPI
```
{
    "productId": "drinks",
    "requestTimestamp": "2019-05-31T12:09:27.652Z",
    "stock": []
}
```


## Stock Endpoint
The below are the endpoints that are exposed to GET PUT PATCH Stock!

### GET Stock By Product Id

This endpoint will list all the stock that are available for the given ProductId. This endpoint covers the below scenarios.

#### Description:
* To enable **CONCURRENT REQUEST** for getting the stock for the productId. If stock is found for the productId, then populate response header with key **ETAG** and value for the key with **VERSION** column value from stock table for the given productId.
* The requested stock for the given productid will be searched from the inmemory **CACHE**, if available the data will be served to the user else it will be searched from the stock table.
* If there are **NO STOCK AVAILABLE** for the productId, then I have populated an attribute called StockMessage which has message as 'There are no stock available for this productId'.
* Check if the user have given valid productId for the stock. If it is **INVALID PRODUCTID**, then respond user as ErroneousJsonException with message as 'Product not found'.

### Getting stock for valid productid

#### Request - Valid ProductId

```
GET /commercetools/api/v1/stock?productId=veg HTTP/1.1
Host: localhost:8080
Content-Type: application/json
Authorization: Basic dXNlcjpwYXNzd29yZA==
```

#### Response - Valid ProductId

**Response Body**
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

**Response Header**
```
ETag: "20"
```

### Getting stock for invalid productid

#### Request - InValid ProductId

```
GET /commercetools/api/v1/stock?productId=cake HTTP/1.1
Host: localhost:8080
Content-Type: application/json
Authorization: Basic dXNlcjpwYXNzd29yZA==
```

#### Response - InValid ProductId

**Response Body**
```
{
    "message": "Invalid Request :: Reason - Product not found",
    "statusCode": "BAD_REQUEST"
}
```

### UPDATE existing stock
This endpoint will help update existing stock.

#### Description:
* If user provided a invalid productId in the request JSON, then respond user as ErroneousJsonException with message as 'Invalid Request :: Reason - Product not found'.
* If user provided a invalid stockId in the request JSON, then respond user as ErroneousJsonException with message as 'Invalid Request :: Reason - Invalid stockid'.
* If user provided a timestamp which for an existing stock with an older timestamp to update, then respond with HTTP status as 204 No Content.
* If a valid JSON request is provided, Since I have implemented catche, the catched values are refreshed so that the GET request can use the data from catche to service.

**Special Case**
* To Enable Concurrent requests for getting the stock for an productId. 
>> If a Stock is found for a productId, The request HTTP header is checked for key 'if-Match' and the value is checked if the value matches the current database version for the stock.
>> If match, the stock is updated
>> else, http status precondition failed exception is thrown to user. 

### Update stock for Invalid JSON (One Sample Request)





## Extract Feature enabled:
The below are the add-on as feature.
* Swagger-UI
* Docker
