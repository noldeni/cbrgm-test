---
title: "RESTful API Design for Microservices - Lessons Learned"
date: 2017-12-09T19:48:27+01:00
draft: false
tags: [ "Development", "Programming", "REST", "Software Engineering", "Microservices", "Representational State Transfer"]
categories:
  - "Development"
  - "Software Engineering"
description: "In this article I would like to conduct a small review and give some tips and tricks for the design of microservices interfaces."
---

Everybody is talking about microservices at the moment. A reasonable interface definition is important for the development of microservices. Together with a few friends, we have developed microservices at the University as part of the "Software Development for the Cloud" event. The event was a great opportunity to familiarize oneself with the topic.

In this article I would like to conduct a small review and give some tips and tricks for the design of microservices interfaces. Maybe it will help one or the other who is facing the same task as me, namely to build a REST API without having any previous knowledge in the field.

## Tips for designing resources

### How to define my resource urls?

When we work with resources, it is usually a single resource or a collection. For this reason, you should create two URLs for each resource.

```bash
/order         # URL for collection
/orders/18     # URL for single element
```

### How to name resource attributes (JSON Attributes)?

When we defined our resources, we thought long and hard about how to name the attributes. We wrote everything in lowercase, but this limited the readability. We finally agreed on the CamelCase notation.

```bash
{"orderStatus": "complete"}
{"orderId": 32123}
```

Do not use underscores or capital letters.

```bash
{"order_status": "complete"} # dont use _
{"order_id": 32123} # dont use _
```

```bash
{"OrderStatus": "complete"} # dont start Uppercase
{"OrderId": 32123} # dont start Uppercase
```

When your REST API is used by a client written in JavaScript. The client converts the JSON response to a JavaScript object and calls its attributes. It is a good idea to stick to the JavaScript convention, which makes the JavaScript code more readable and intuitive.

## Tips for designing the API

### Should version and format be in URL?

Versioning your API is a good idea. We tagged the version of our API with the prefix' v'. Move it all the way to the left in the URL so that it has the largest scope (e. g. /v1/orders). Use a simple ordinal number.

```bash
/v1/orders
```

```bash
/api/v1/orders
```

#### What about versions like 1.1, 1.2, ...

Don't use point notation like v1.1 because it implies a granularity of versioning that doesn't work well with APIs - it's an interface and not an implementation. Stick with v1, v2, and so on!

#### Using a subdomain?

If you have access to your webservers configuration files, you can move your api to a subdomain. A common subdomain used for accessing your API gateway is `"api.domain.com"`.

### How to use http methods?

Don't use verbs, use Nouns instead! Many APIs start by using a method-driven approach to URL design. These method-based URLs sometimes contain verbs, sometimes at the beginning, sometimes at the end.

Don't do:

```bash
/getAllOrders
/getAllCompletedOrders
/createOrder
/updateOrder
```

Instead you should use http methods like this to enable CRUD operations:

```bash
GET /orders # get a resource
GET /orders?status=complete # get a resource with specific attribute(s)
POST /orders # create a resource
PUT /orders/18 # update a resource
```

The resource should be in the foreground when designing the API. The operation on a resource should always be encapsulated by the resource itself.

```bash
GET /orders?status=complete # get a resource with specific attribute(s)
```

As shown above, you should use the `? ` to enable complex queries. The base url of your resource should remain small and narrow.

### How to deal with relationships in data?

In our API we have defined an order resource. Orders have certain items and it should be possible to access the items within the resource.

```bash
{
  "orderId":3212,
  "orderStatus":"complete",
  "items": ["Milk", "Eggs", "Chocolate"]
}
```
Relationships can be complex. Nevertheless, we want to keep our URL as simple as possible. To get to the list of our items we use the following format at Resources:

```bash
/resource/identifier/resource # template

/orders/18/items # access item list
```

### How to use http status codes?

Http status codes allow us to provide appropriate feedback on a client request. There are over 70 different status codes, but we only need a handful of them to inform clients about specific issues.

Basically, we can limit the status codes to three basic situations:

* Sucessful - Everything is OK
* Client Error - The application did something wrong (Wrong request format for example..)
* Server Error - Our API did something wrong or processed the request in a wrong way

We started designing our API with only three Statuscodes and ended up with 8 in total.

* 200 OK
* 400 Bad Request
* 500 Internal Server Error

You can go beyond that and use:

* 201 Created
* 304 Not Modified
* 404 Not Found
* 401 Unauthorized
* 403 Forbidden

It is important that the code that is returned can be consumed and acted upon by the application's business logic -  for example, in a Case- or if-Statement.

Therefore, provide useful error responses like:

```bash
# 400 Bad Request
{
"message": "You submitted an invalid state. Valid state values are 'internal' or 'external'",
"errorCode": 352,
"additionalInformation" : "http://www.domain.com/rest/errorcode/352"
}
```

### How to deal with data intensive queries?

For data-intensive queries via the API interface it is useful to provide pagination. It is common to use the parameters offset and limit, which are well-known from databases. Don't forget to implement a defaul query like this:

```bash
/orders       # returns the orders 0 - 10
```

Allow to query a specific offset and limit, where offset means where to start the query and limit how many resources will be returned:

```bash
/orders?offset=50&limit=20       # returns the orders 50 to 70
```

### How to deal with Non-Resource Responses?

Sometimes requests must be provided to the API that are not directly associated with a specific resource. The value added tax must be charged for our orders. The calculation cannot be assigned directly to a particular order, but is a simple calculation.

**Use verbs not nouns for that!**

Here's and example how to calculate the tax based on order sum  and percentage:

```bash
/calculate?orderSum=123.23&percentage=19.00
```

Make it clear in your API documentation that these “non-resource” scenarios are different.  

### How to implement searching for resource?

While the search for a certain resource is easy to implement, we need a different design approach for a multi-resource search.

**Global search**

```bash
/search?q=food+drinks
```
Search ist the verb and ?q represents the query.

**Scoped search**

To add scope to your search, you can prepend with the scope of the search. For example,search in orders owned by resource ID 5678

```bash
/orders/5678/items?q=food+drinks
```

## Tips for Security

### Use OAuth 2.0

Use the latest and greatest OAuth - OAuth 2.0 (as of this writing). It means that Web or mobile apps that expose APIs don’t have to share passwords. It allows the API provider to
revoke tokens for an individual user, for an entire app, without requiring the user to change their original password. This is critical if a mobile device is compromised or if a rogue app is discovered.  

## Tips for API documentation

The documentation of the REST API is one of the most important ones! Your interface can only be used if it is well documented. In our project we have made the mistake of underestimating the subject of documentation and have only sporadically communicated request formats, URL calls and the like in slack or via email. However, once a change has been made or a resource extended, the rest of the team usually knew nothing about it.

The documentation can be implemented using tools. The best known way to do this is Swagger. io, but also markdown files in combination with a static website generator are a possibility.

# Conclusion

I hope I could help one or the other with this article and answer a few questions. Eventually I will add more content to this article if I have more experience in the field. Until then, thanks for reading!
