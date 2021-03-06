# HTTP 动词
HTTP 动词包含了REST六大约束之一的“接口统一”约束的主要部分，并为我们提供了基于名词的资源与之相对应的动作。最常用的或者说最主要的HTTP 动词（或调用方法）是POST、GET、PUT和DELETE，它们分别对应于创建，读取，更新和删除（或CRUD）操作。当然，还有许多其他HTTP 动词，但使用频率较低，在那些不常用的方法中，OPTIONS和HEAD又比其他方法更频繁地被使用。


## <div id="get">GET</div>
HTTP GET方法常被用于检索（或读取）资源。在正常（或非错误）路径中，GET通过返回XML或JSON返回请求的资源，以及通过返回HTTP响应吗200来表示请求OK。在错误情况下，它通常返回404（NOT FOUND）或400（BAD REQUEST）。GET的URI举例如下：
```
    GET http://www.example.com/customers/12345
    GET http://www.example.com/customers/12345/orders
    GET http://www.example.com/buckets/sample
```
根据HTTP设计的规范，GET（以及HEAD）请求仅用于读取数据而不是更改它。因此，当以这种方式请求资源时，这些操作被认为是安全的。也就是说，可以调用它们而不用担心数据被篡改或删除的风险 - 调用一次操作与调用十次操作，产生的效果最终是相同的。这样表达的正式说法是，GET（和HEAD）是幂等的，这意味着多个相同的请求最终产生的结果会与单个请求产生相同的结果。  
永远不要通过GET去进行不安全的操作，即它永远不应该去篡改或删除服务器上的任何资源。


## <div id="put">PUT</div>
PUT是更新时最常用的HTTP方法，通过将原始资源更新后的表示包含在请求体内，将资源PUT到已知URI来完成更新操作。但是，在客户端而不是服务器创建资源ID的情况下，PUT也可用于创建资源，换句话说，如果PUT的URI中不包含的请求体中资源的ID，那么该资源将被新建。同样情况下，POST请求体中也会包含资源的表示。许多人都觉得这样会很令人费解和困惑。因此，如果不是特别需要的话，应该谨慎使用PUT方法创建资源。或者，使用POST方式创建新资源并在正文中提供客户端定义的ID表示亦或者可能是一个不包含ID的资源（参见下面的POST）。PUT的URI举例如下：
```
    PUT http://www.example.com/customers/12345
    PUT http://www.example.com/customers/12345/orders/98765
    PUT http://www.example.com/buckets/secret_stuff
```
成功更新后，会从PUT请求返回状态码200（如果返回正文中没有任何内容或者返回中没有正文，则返回204）。如果使用PUT进行资源创建，则在成功创建时返回HTTP状态201。响应中的body是可选的，但是如果提供一个响应体将会消耗更多的带宽。另外，由于客户端已经设置了资源ID，因此无需在创建后的响应中通过Location头返回资源的链接。请参阅下面的返回值部分。  
PUT不是一个安全的操作，因为它会修改（或创建）服务器上的资源，但它是幂等的。 换句话说，如果您使用PUT创建或更新资源，然后再次进行相同的调用，则资源仍然存在，并且仍然具有与第一次调用结束时相同的状态。但是，如果在资源上调用PUT会触发资源的修改计数器，则该调用不再是幂等的。有时会发生这种情况，并且这样就可能足以证明调用不是幂等的。但是，建议保持PUT请求的幂等性，并且，强烈建议对非幂等请求使用POST方法。


## <div id="post">POST</div>
POST是最常用的创建资源的HTTP动词，特别是，它常被用于创建附加资源。也就是说新创建的资源从属于某些资源，换句话讲，创建资源后将资源POST到服务器并将其与上级资源关联并分配ID以及资源的访问URI等等。例如：
```
    POST http://www.example.com/customers
    POST http://www.example.com/customers/12345/orders
```
资源成功创建后，返回HTTP状态201，其Header中含有Location链接，该链接指向具有201 HTTP状态的新创建资源。POST既不安全也不是幂等。 因此，建议用于非幂等资源请求。 发起两个相同的POST请求最有可能导致两个资源包含相同的信息。


## <div id="putvspost">创建时使用PUT vs POST？</div>
简而言之，尽量使用POST进行资源创建，否则会由客户端在PUT时负责决定新资源将使用的URI（通过它的资源名称或ID）；换言之，如果客户端知道资源要使用的URI（或资源ID）是什么，则可以使用PUT创建资源并给定该URI，否则，应使用POST创建资源并让服务器或服务负责决定新创建的资源的URI。当客户端在创建之前不知道（也不应该知道）结果资源URI将是什么时，应该使用POST来创建新资源。


## <div id="delete">DELETE</div>
DELETE很容易理解，它常被用于删除由URI标识的资源。例如：
```
    DELETE http://www.example.com/customers/12345
    DELETE http://www.example.com/customers/12345/orders
    DELETE http://www.example.com/buckets/sample
```
成功删除后，返回HTTP状态200（OK）以及响应正文，正文可能是已删除项目的表示（这通常需要很多带宽），或者可能是包装响应（请参阅下面的返回值）。要么是返回HTTP状态204（NO CONTENT）而没有响应正文，但通常来讲，建议的响应是204状态，没有正文，或JSON样式响应和HTTP状态200。需要尤其指出，DELETE操作是幂等的。如果是删除资源，则将其删除。在该资源上反复调用DELETE最终结果相同：资源消失了。 如果调用DELETE会触发递减计数器（在资源内），则DELETE调用不再是幂等的。如前所述，只要没有资源数据被改变，就可以更新使用统计和测量，同时仍然考虑服务幂等。使用POST作为非幂等资源建议请求。但是，有一个关于DELETE幂等性的警告。 第二次在资源上调用DELETE通常会返回404（NOT FOUND），因为它已被删除而不再可查找。 这使得DELETE操作不再是幂等的，这通常是在从数据库中删除资源而不是简单地将资源标记为已删除。  

下面的表总结了主要HTTP方法与资源URI结合使用时的建议返回值：

| HTTP动词 | /customers | /customers/{id} | 
|--------|--------|--------| 
|  GET   | 200(OK)，customer的列表。通过使用分页，排序和过滤来获取列表 | 200(OK)，表示成功获取单个用户信息。404(NOT FOUND)，表示没找到或者ID无效 | 
|  PUT   | 404（NOT FOUND），除非开发者想要更新或替换整个资源集 | 200（OK）或204（NO Content）。 404（NOT FOUND），表示找不到或ID无效 | 
|  POST  | 201(Created)，在返回的Header里面的Location字段存储有指向新增资源的URI，该URI中包含新ID | 404(NOT FOUND) | 
| DELETE | 404（NOT FOUND），除非你想要删除整个集合 - 这通常不太可取。 | 200（OK）。 404（NOT FOUND），表示找不到或ID无效 |
































