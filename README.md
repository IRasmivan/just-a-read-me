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
cache-control: no-cache
Postman-Token: a1cb6b97-b1e3-4132-9dd4-a776f6d1c78b
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
```


## Extract Feature enabled:
The below are the add-on as feature.
* Swagger-UI
* Docker
