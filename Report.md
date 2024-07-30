Research on RESTful API
Table of contents
•	Introduction
What is an API
RESTful APIs
•	Design & Implementation
Business resources and APIs
HTTP methods
Asynchronous operations
Filtering & Pagination
Partial responses
HATEOAS navigation
Versioning
Exception Handling
Caching
•	Integration
Publishing
Documentation
Client SDK
Testing
Monitoring
•	References
Introduction
What is an API?
APIs, or Application Programming Interfaces, are tools that allow two software components to interact with one another using specified definitions and protocols. For instance, the weather bureau's software system holds daily weather information. The weather app on your phone accesses this data through APIs to display daily weather updates. In this context, "Application" refers to any software with a specific function, and "Interface" is like a service contract between two applications, outlining how they communicate using requests and responses.
RESTful APIs
Representational State Transfer (REST) is an architectural style that sets constraints for creating web services. A REST API allows access to web services in a straightforward and flexible manner, without requiring any processing. REST is typically favored over the more complex Simple Object Access Protocol (SOAP) because it consumes less bandwidth and is simpler and more adaptable, making it more suitable for internet use. REST APIs are used to retrieve or provide information from a web service, and all communication through REST APIs is conducted using HTTP requests only.
Design & Implementation
Here are some of the main general design principles of RESTful APIs using HTTP which we will go into more details going on:
•	REST APIs are designed around resources, which are any kind of object, data, or service that can be accessed by the client.
•	A resource has an identifier, which is a URI that uniquely identifies that resource. For example, the URI for a particular customer order might be:
 	https://emarket.com/orders/1
•	Clients interact with a service by exchanging representations of resources. Many web APIs use JSON as the exchange format. For example, a GET request to the URI listed above might return this response body:
 	JSON
 	{"orderId":1,"orderValue":99.90,"productId":1,"quantity":1}
•	REST APIs use a uniform interface, which helps to decouple the client and service implementations. For REST APIs built on HTTP, the uniform interface includes using standard HTTP verbs to perform operations on resources. The most common operations are GET, POST, PUT, PATCH, and DELETE.
•	REST APIs use a stateless request model. HTTP requests should be independent and might occur in any order, so keeping transient state information between requests is not feasible. The only place where information is stored is in the resources themselves, and each request should be an atomic operation. This constraint enables web services to be highly scalable, because there is no need to retain any affinity between clients and specific servers. Any server can handle any request from any client.
•	REST APIs are driven by hypermedia links that are contained in the representation.
Organize the API design around resources
Focus on the business entities exposed by the web API. For instance, in an e-commerce system, the primary entities might be customers and orders. Creating an order would involve sending an HTTP POST request with the order information, and the HTTP response would indicate whether the order was successfully placed.
A resource doesn't need to correspond to a single physical data item. For example, an order resource might be represented internally by several tables in a relational database but presented to the client as a single entity. Avoid creating APIs that mirror the internal database structure. REST is intended to model entities and the operations that can be performed on them, without exposing the internal implementation details to the client.
Entities are often organized into collections (such as orders and customers). A collection is a separate resource from its individual items and should have its own URI. For example, the URI `https://emarket.com/orders` might represent the collection of orders. An HTTP GET request to this URI retrieves a list of items in the collection, with each item having its own unique URI. An HTTP GET request to an item's URI returns the details of that item.
Use a consistent naming convention for URIs, generally favoring plural nouns for collections. Organize URIs for collections and items into a hierarchy, such as `/customers` for the customers collection and `/customers/5` for the customer with ID 5. This practice helps keep the web API intuitive.
Consider the relationships between different types of resources and how to expose these associations. For example, `/customers/5/orders` could represent all orders for customer 5, while `/orders/99/customer` might represent the customer associated with order 99. However, extending this model too far can become cumbersome. A better approach is to include navigable links to associated resources within the HTTP response body, as detailed in the section on HATEOAS.
Additionally, minimize the load on the web server by avoiding "chatty" web APIs that expose numerous small resources, requiring multiple requests to gather all needed data. Instead, denormalize the data and combine related information into larger resources that can be retrieved with a single request. Balance this approach against the potential overhead of fetching unnecessary data, as retrieving large objects can increase latency and bandwidth costs.
Lastly, avoid tightly coupling the web API to the underlying data sources. For example, don’t expose each database table as a collection of resources. Treat the web API as an abstraction layer over the database, and if needed, introduce a mapping layer between the database and the API. This way, client applications remain insulated from changes to the database schema.
Define API operations in terms of HTTP methods
The HTTP protocol defines a number of methods that assign semantic meaning to a request. The common HTTP methods used by most RESTful web APIs are:
•	GET retrieves a representation of the resource at the specified URI. The body of the response message contains the details of the requested resource.
•	POST creates a new resource at the specified URI. The body of the request message provides the details of the new resource. Note that POST can also be used to trigger operations that don't actually create resources.
•	PUT either creates or replaces the resource at the specified URI. The body of the request message specifies the resource to be created or updated.
•	PATCH performs a partial update of a resource. The request body specifies the set of changes to apply to the resource.
•	DELETE removes the resource at the specified URI.
Ensure that each request is stateless.
Each request should be treated as independent and atomic, with no dependencies on previous or subsequent requests from the same client. This approach enhances scalability, as the web service can be deployed across multiple servers. Client requests can be directed to any of these servers, and the results should be consistent. It also improves availability because if a web server fails, requests can be routed to another instance without affecting client applications while the failed server is restarted.
Conform to HTTP Semantics
When designing an API, it is essential to follow HTTP specifications. Here are some typical considerations for various HTTP methods:
GET Methods
-Successful Request: Returns HTTP status code 200 (OK).
- Resource Not Found: Returns HTTP status code 404 (Not Found).
- No Content: If the request is fulfilled but there is no response body, return HTTP status code 204 (No Content). For example, a search with no matches could use this status.
POST Methods
- Resource Creation: Returns HTTP status code 201 (Created). The URI of the new resource should be included in the `Location` header, and the response body should contain a representation of the resource.
- Processing Without Resource Creation: Returns HTTP status code 200 (OK) with the result of the operation in the response body. If there is no result to return, HTTP status code 204 (No Content) may be used instead.
- Invalid Data: If the request contains invalid data, return HTTP status code 400 (Bad Request). Include additional error information or a link to a URI with more details in the response body.
PUT Methods
- Resource Creation: Returns HTTP status code 201 (Created), similar to POST.
- Resource Update: Returns HTTP status code 200 (OK) or 204 (No Content). If updating an existing resource is not possible, consider returning HTTP status code 409 (Conflict).
- Bulk Operations: Implement bulk PUT operations to update multiple resources in a collection. The request should specify the collection URI, and the request body should detail the resources to be updated. This helps reduce chattiness and improve performance.
DELETE Methods
- Successful Deletion: Returns HTTP status code 204 (No Content), indicating successful handling with no further information in the response body.
- Resource Not Found: Returns HTTP status code 404 (Not Found) if the resource does not exist.
Idempotency of GET, PUT, DELETE, HEAD, and PATCH
- Idempotency: These actions should be idempotent, meaning that repeating the same request on the same resource should result in the same state. For example, multiple DELETE requests to the same URI should have the same effect, although the status codes might differ. The first DELETE request may return 204 (No Content), while subsequent DELETE requests could return 404 (Not Found).
Asynchronous operations
When a POST, PUT, PATCH, or DELETE operation requires significant processing time, waiting for completion before responding can lead to unacceptable latency. In such cases, consider making the operation asynchronous. Return HTTP status code 202 (Accepted) to indicate that the request has been accepted for processing but is not yet complete.
Additionally, expose an endpoint that allows clients to check the status of the asynchronous request. Include the URI of this status endpoint in the `Location` header of the 202 response, so clients can monitor the progress by polling this status endpoint. For example:
 
If the client sends a GET request to this endpoint, the response should contain the current status of the request. Optionally, it could also include an estimated time to completion or a link to cancel the operation.
  If the asynchronous operation creates a new resource, the status endpoint should return status code 303 (See Other) after the operation completes. In the 303 response, include a Location header that gives the URI of the new resource.
Empty sets in message bodies
Any time the body of a successful response is empty, the status code should be 204 (No Content). For empty sets, such as a response to a filtered request with no items, the status code should still be 204 (No Content), not 200 (OK).
Filter and paginate data
Exposing a collection of resources through a single URI can result in applications retrieving large amounts of data when only a subset is needed. For example, if a client application needs to find all orders with a cost exceeding a certain value, retrieving all orders from `/orders` and filtering on the client side is inefficient. This approach wastes network bandwidth and server processing power.
Instead, design the API to support filtering through query strings, such as `/orders?minCost=n`. The web API should parse and handle the `minCost` parameter in the query string and return only the filtered results from the server side.
To manage potentially large response sizes from GET requests over collection resources, implement features to limit the amount of data returned by a single request. Consider supporting query strings that specify the maximum number of items to retrieve and a starting offset within the collection. This approach helps control the data volume and improves efficiency. For example:
 
Also consider imposing an upper limit on the number of items returned to prevent potential Denial of Service attacks. For GET requests that return paginated data, include metadata indicating the total number of resources available in the collection. This helps clients understand the extent of the data set.
Additionally, if each item contains a large amount of data, extend the approach to limit the fields returned. Use a query string parameter that accepts a comma-delimited list of fields, such as `/orders?fields=ProductID,Quantity`, to allow clients to request only the specific fields they need.
Support partial responses for large binary resources
Support partial responses for large binary resources. For resources with large binary fields, such as files or images, consider allowing retrieval in chunks to address issues with unreliable connections and improve response times. Implement the `Accept-Ranges` header in GET requests to indicate that the API supports partial requests. This allows client applications to request specific byte ranges of a resource, rather than retrieving the entire resource at once.
The response message indicates that this is a partial response by returning HTTP status code 206. The Content-Length header specifies the actual number of bytes returned in the message body (not the size of the resource), and the Content-Range header indicates which part of the resource this is (bytes 0-2499 out of 4580):
 
A subsequent request from the client application can retrieve the remainder of the resource. This ensures that the requests are still atomic and stateless.
Use HATEOAS (Hypertext As The Engine Of Application State) to enable navigation to related resources.
REST aims to allow navigation through the entire set of resources without prior knowledge of the URI scheme. Each HTTP GET request should return information to locate related resources via hyperlinks included in the response. Additionally, provide details on the operations available for each of these resources. This approach makes the system act as a finite state machine, where each response contains the information needed to transition from one state to another, requiring no additional information.
Versioning a RESTful web API
A web API is unlikely to remain static as business requirements evolve. New collections of resources may be added, relationships between resources might change, and resource structures could be modified. While updating a web API to meet new requirements is typically straightforward, it’s crucial to consider how these changes impact client applications that consume the API. Unlike the API developer, who controls the API, client applications may be developed by third-party organizations and are not under the same level of control.
The goal is to ensure that existing client applications continue to function without changes while allowing new client applications to leverage new features and resources. Versioning is a key strategy to manage this. It allows the API to specify which features and resources are available in each version, enabling client applications to request a specific version of the API. Various versioning approaches offer different benefits and trade-offs, which are discussed in the following sections.
No Versioning
This approach is the simplest and might work for some internal APIs. In this model, significant changes are handled by introducing new resources or links, rather than versioning the API. Adding new content to existing resources generally doesn’t break existing clients if they can ignore unrecognized fields. New client applications can take advantage of these new fields.
However, radical changes to the schema, such as removing or renaming fields, or altering relationships between resources, could break existing client applications. In these cases, versioning or other strategies should be considered to manage these breaking changes.
URI versioning
In URI versioning, each time the web API or resource schema is modified, you add a version number to the URI of each resource. For instance, `/v1/orders` could be the endpoint for the initial version, while `/v2/orders` would represent the updated version. The previous URIs should still function as they did before, returning data according to their original schema.
This approach is straightforward but can become cumbersome as the API evolves and needs to support multiple versions. Additionally, from a purist's perspective, the URI should represent the resource itself (e.g., customer 3) rather than its version. This scheme can also complicate HATEOAS implementation, as all links will need to include the version number in their URIs.
Handling exceptions
Consider the following points if an operation throws an uncaught exception.
Capture exceptions and return a meaningful response to clients
The code implementing an HTTP operation should include comprehensive exception handling to prevent uncaught exceptions from propagating to the framework. If an exception prevents the operation from completing successfully, it should be reflected in the response message with a meaningful description of the error and an appropriate HTTP status code. Avoid using status code 500 (Internal Server Error) for all errors.
For example, if a user request leads to a database constraint violation (such as attempting to delete a customer with outstanding orders), return status code 409 (Conflict) with a message explaining the conflict. If the request cannot be fulfilled due to other issues, return status code 400 (Bad Request). Refer to the full list of HTTP status codes on the W3C website for more details.
This code example traps different conditions and returns an appropriate response:  
Handle Exceptions Consistently and Log Information About Errors
To manage exceptions effectively, implement a global error handling strategy across the web API. This ensures that all exceptions are handled in a consistent manner. Additionally, integrate error logging to capture detailed information about each exception, but ensure that this log is not accessible to clients over the web.
Distinguish Between Client-Side and Server-Side Errors.
The HTTP protocol categorizes errors into client-side (HTTP 4xx status codes) and server-side (HTTP 5xx status codes) errors. Ensure that your error responses adhere to this distinction, clearly differentiating between issues caused by the client application and those arising from server-side problems.
Support client-side caching
In a distributed environment with web servers and client applications, network traffic can become a significant bottleneck, especially with frequent requests or large data transfers. To address this, aim to minimize the amount of data transmitted across the network.
The HTTP 1.1 protocol supports caching through the `Cache-Control` header, which can be used by clients and intermediate servers to cache responses. When a client sends an HTTP GET request, the response can include a `Cache-Control` header that specifies whether the response data can be cached, how long it can be stored, and when it should be considered outdated. This helps reduce the amount of traffic and improves efficiency by avoiding redundant data transfers.
The following example shows an HTTP GET request and the corresponding response that includes a Cache-Control header:  
In this example, the Cache-Control header specifies that the data returned should be expired after 600 seconds, and is only suitable for a single client and must not be stored in a shared cache used by other clients (it is private). The Cache-Control header could specify public rather than private in which case the data can be stored in a shared cache, or it could specify no-store in which case the data must not be cached by the client.
Integration
Publishing and managing a web API
To make a web API available for client applications, it must be deployed to a host environment, typically a web server, though it could be another type of host process. Consider the following points when publishing a web API:
•	**Authentication and Authorization:** Ensure all requests are authenticated and authorized, enforcing the appropriate level of access control.
–	**Quality Guarantees:** For commercial web APIs with quality guarantees, ensure the host environment is scalable to handle varying loads over time.
–	**Request Metering:** Implement request metering for monetization purposes if necessary.
–	**Traffic Regulation:** Regulate traffic flow to the web API and implement throttling for clients that have exhausted their quotas.
–	**Regulatory Requirements:** Comply with regulatory requirements by logging and auditing all requests and responses.
–	**Server Health Monitoring:** Monitor the health of the server hosting the web API and restart it if necessary to ensure availability.
Document the REST operations for a web API
Developers constructing client applications typically require information on how to access the web API, and documentation concerning the parameters, data types, return types, and return codes that describe the different requests and responses between the web service and the client application.
This documentation should provide:
•	Documentation for the product, listing the operations that it exposes, the parameters required, and the different responses that can be returned. Note that this information is generated from the details provided in step 3 in the list in the Publishing a web API by using the Microsoft Azure API Management Service section.
•	Code snippets that show how to invoke operations from several languages, including JavaScript, C#, Java, Ruby, Python, and PHP.
•	A developers' console that enables a developer to send an HTTP request to test each operation in the product and view the results.
•	A page where the developer can report any issues or problems found.
Implement a client SDK
Building a client application that makes REST requests to access a web API involves writing extensive code to construct and properly format each request, send it to the server hosting the web service, and parse the response to determine success or failure and extract any returned data. To simplify this
process for the client application, you can provide an SDK that wraps the REST interface, abstracting these low-level details within a more functional set of methods. The client application uses these methods, which transparently convert calls into REST requests and then convert the responses back into method return values. This approach is commonly used by many services, including the Azure SDK.
Creating a client-side SDK is a significant task, requiring consistent implementation and careful testing. However, much of this process can be automated, and many vendors provide tools to facilitate these tasks.
Testing a web API
A web API should be tested as thoroughly as any other piece of software. Consider creating unit tests to validate its functionality. The nature of a web API introduces additional requirements to ensure it operates correctly. Pay particular attention to the following aspects:
- Test all routes to verify they invoke the correct operations. Be especially aware of HTTP status code 405 (Method Not Allowed), which can indicate a mismatch between a route and the HTTP methods (GET, POST, PUT, DELETE) assigned to that route. Send HTTP requests to routes that don't support them, such as submitting a POST request to a specific resource when POST requests should only be sent to resource collections. The valid response should be status code 405 (Not Allowed).
- Verify that all routes are properly protected and subject to the appropriate authentication and authorization checks. While aspects like user authentication are likely managed by the host environment, security tests should still be part of the deployment process.
- Test exception handlingto ensure each operation returns appropriate and meaningful HTTP responses to the client application.
-Verify that request and response messages are well-formed. For example, confirm that an HTTP POST request containing data for a new resource in x-www-form-urlencoded format is correctly parsed, creates the resource, and returns a response with the new resource details, including the correct Location header.
- Check all links and URIs in response messages.** Ensure that an HTTP POST message returns the URI of the newly created resource and that all HATEOAS links are valid.
- Ensure each operation returns the correct status codes** for various input combinations. For instance:
- Successful queries should return status code 200 (OK).
- A missing resource should return status code 404 (Not Found).
- A successful deletion request should return status code 204 (No Content).
- A successful creation request should return status code 201 (Created).
-Watch for unexpected status codes in the 5xx range, indicating the host server's inability to fulfill a valid request.
- Test different request header combination to ensure the web API returns the expected information in response messages.
-Test query strings. For operations that can take optional parameters (like pagination requests), test different combinations and orders of parameters.
- Verify that asynchronous operations complete successfully. If the web API supports streaming for large binary objects (like video or audio), ensure client requests are not blocked during data streaming. If the web API uses polling for long-running data modification operations, confirm that these operations report their status correctly as they proceed.
-You should also create and run performance tests to check that the web API operates satisfactorily under duress. You can build a web performance and load test project by using Visual Studio Ultimate.
Monitoring a web API
Depending on how you publish and deploy your web API, you can either monitor it directly or gather usage and health information by analyzing traffic through an API Management service. When deploying the web API, all traffic should be examined, and the following statistics should be collected:
•	Server response time.
–	Number of server requests and details of each request.
–	Top slowest requests by average response time.
–	Details of any failed requests.
–	Number of sessions initiated by different browsers and user agents.
–	Most frequently viewed pages.
–	Different user roles accessing the web API.
Track clients and implement throttling to reduce the chances of DoS attacks
If a particular client makes numerous requests within a short period, it might monopolize the service and degrade performance for other clients. To address this, a web API can monitor calls from client applications by tracking the IP addresses of incoming requests or logging each authenticated access. This information can be used to limit resource access. If a client exceeds a defined limit, the web API can respond with a status 503 (Service Unavailable) message and include a `Retry-After` header specifying when the client can send the next request without being rejected. This approach helps reduce the risk of a Denial of Service (DoS) attack by preventing a set of clients from overloading the system.
Manage persistent HTTP connections carefully
The HTTP protocol supports persistent connections when available. The HTTP 1.0 specification introduced the `Connection: Keep-Alive` header, allowing a client to inform the server that it can use the same connection for subsequent requests rather than opening new ones. If the client does not reuse the connection within a time period defined by the host, the connection will automatically close. This behavior is the default in HTTP 1.1, as used by Azure services, so there is no need to include `Keep-Alive` headers in messages.
Maintaining an open connection can enhance responsiveness by reducing latency and network congestion. However, it can negatively impact scalability by keeping unnecessary connections open longer than needed, limiting other concurrent clients' ability to connect. It can also affect battery life on mobile devices; if an application makes occasional requests, keeping the connection open can drain the battery more quickly. To prevent a persistent connection in HTTP 1.1, the client can include a `Connection: Close` header with messages to override the default behavior. Similarly, if a server is handling a large number of clients, it can include a `Connection: Close` header in response messages to close the connection and conserve server resources.
References
•	W3C
•	Microsoft
•	Geeks For Geeks
