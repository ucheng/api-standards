# 美國白宮Web API標準 
White House Web API Standards

* [指導原則(Guidelines)](#guidelines)
* [Pragmatic REST](#pragmatic-rest)
* [RESTful URLs](#restful-urls)
* [HTTP 方法](#http-verbs)
* [回應(Response)](#responses)
* [錯誤處理(Error Handling)](#error-handling)
* [版本(Versions)](#versions)
* [資料量限制(Record Limits)](#record-limits)
* [請求與回應範例(Request & Response Examples)](#request-response-examples)
* [模擬回應(Mock Responses)](#mock-responses)
* [JSONP](#jsonp)

## <a name="guidelines"></a>指導原則

**這份文件提供了White House Web APIs的準則和範例，來提倡API的一致性，可管理性以及橫跨各種應用的最佳實務。White House APIs目標在於提供一個真正的Restful API介面以及良好的開發者體驗。**   
This document provides guidelines and examples for White House Web APIs, encouraging consistency, maintainability, and best practices across applications. White House APIs aim to balance a truly RESTful API interface with a positive developer experience (DX). 


這份文件從以下的連結引用了相當多的內容:
* [Designing HTTP Interfaces and RESTful Web Services](http://munich2012.drupal.org/program/sessions/designing-http-interfaces-and-restful-web-services)
* [API Facade Pattern](http://apigee.com/about/content/api-fa%C3%A7ade-pattern), by Brian Mulloy, Apigee
* [Web API Design](http://pages.apigee.com/web-api-design-ebook.html), by Brian Mulloy, Apigee
* [Fielding's Dissertation on REST](http://www.ics.uci.edu/~fielding/pubs/dissertation/top.htm)

## Pragmatic REST

These guidelines aim to support a truly RESTful API. Here are a few exceptions:
* Put the version number of the API in the URL (see examples below). Don’t accept any requests that do not specify a version number.
* Allow users to request formats like JSON or XML like this:
    * http://example.gov/api/v1/magazines.json
    * http://example.gov/api/v1/magazines.xml

## RESTful URLs

### General guidelines for RESTful URLs
* **一個URL標示著一個資源(resource)。** A URL identifies a resource.
* **URL應該包含名詞而非動詞。** URLs should include nouns, not verbs.
* **使用複數名詞來維持一致性(不要用單數名詞)。** Use plural nouns only for consistency (no singular nouns).
* **使用HTTP方法(GET, POST,PUT, DELETE)來操作資料。**Use HTTP verbs (GET, POST, PUT, DELETE) to operate on the collections and elements.
* **資源的檢索最多兩層。**You shouldn’t need to go deeper than resource/identifier/resource. 
* **在你的URL加上版號。**Put the version number at the base of your URL, for example http://example.com/v1/path/to/resource. [(註1)](#note-1)    
* URL v. header:
    * If it changes the logic you write to handle the response, put it in the URL.
    * If it doesn’t change the logic for each response, like OAuth info, put it in the header.
* **指定選擇性欄位並用逗號分開。**Specify optional fields in a comma separated list.
* **資料格式應該如api/v2/resource/{id}.json**  Formats should be in the form of api/v2/resource/{id}.json 

### 良好的URL範例
* 回傳雜誌清單:
    * GET http://www.example.gov/api/v1/magazines.json
* Filtering is a query:
    * GET http://www.example.gov/api/v1/magazines.json?year=2011&sort=desc
    * GET http://www.example.gov/api/v1/magazines.json?topic=economy&year=2011
* 回傳單一雜誌(格式為json):
    * GET http://www.example.gov/api/v1/magazines/1234.json
* 回傳所有屬於這本雜誌的文章:
    * GET http://www.example.gov/api/v1/magazines/1234/articles.json
* 回傳所有屬於這本雜誌的文章(格式為XML)
    * GET http://example.gov/api/v1/magazines/1234/articles.xml
* 指定選擇性欄位併用逗號分隔:
    * GET http://www.example.gov/api/v1/magazines/1234.json?fields=title,subtitle,date
* 新增一篇文章至某本雜誌:
    * POST http://example.gov/api/v1/magazines/1234/articles

### 不好的URL範例
* 非負數名詞:
    * http://www.example.gov/magazine
    * http://www.example.gov/magazine/1234
    * http://www.example.gov/publisher/magazine/1234
* 在URL中使用動詞:
    * http://www.example.gov/magazine/1234/create
* Filter outside of query string
    * http://www.example.gov/magazines/2011/desc

## HTTP Verbs

HTTP verbs, or methods, should be used in compliance with their definitions under the [HTTP/1.1](http://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html) standard.
The action taken on the representation will be contextual to the media type being worked on and its current state. Here's an example of how HTTP verbs map to create, read, update, delete operations in a particular context:

| HTTP METHOD | POST            | GET       | PUT         | DELETE |
| ----------- | --------------- | --------- | ----------- | ------ |
| CRUD OP     | CREATE          | READ      | UPDATE      | DELETE |
| /dogs       | Create new dogs | List dogs | Bulk update | Delete all dogs |
| /dogs/1234  | Error           | Show Bo   | If exists, update Bo; If not, error | Delete Bo |

(Example from Web API Design, by Brian Mulloy, Apigee.)


## Responses

* **索引鍵不應包含值。**No values in keys 
* No internal-specific names (e.g. "node" and "taxonomy term")
* Metadata should only contain direct properties of the response set, not properties of the members of the response set

### 良好的範例

索引鍵不包含值:

    "tags": [
      {"id": "125", "name": "Environment"},
      {"id": "834", "name": "Water Quality"}
    ],


### 不好的範例

索引鍵包含值:

    "tags": [
      {"125": "Environment"},
      {"834": "Water Quality"}
    ],


## Error handling

**錯誤的回應應該包含一個常見的HTTP狀態碼, 提供給開發者的訊息，提供給終端使用者的訊息(當適合的時候)，內部錯誤碼, 以及讓開發者可以找到更多資訊的連結。** 

Error responses should include a common HTTP status code, message for the developer, message for the end-user (when appropriate), internal error code (corresponding to some specific internally determined ID), links where developers can find more info. 
For example:

    {
      "status" : 400,
      "developerMessage" : "Verbose, plain language description of the problem. Provide developers
       suggestions about how to solve their problems here",
      "userMessage" : "This is a message that can be passed along to end-users, if needed.",
      "errorCode" : "444444",
      "moreInfo" : "http://www.example.gov/developer/path/to/help/for/444444,
       http://drupal.org/node/444444",
    }


**使用三個簡單，常見的回應狀態碼來表示(1)成功, (2)因為用戶端的問題而導致的失敗, (3)因為伺服器端的問題而導致的失敗。**
Use three simple, common response codes indicating (1) success, (2) failure due to client-side problem, (3) failure due to server-side problem:

* 200 - OK
* 400 - Bad Request
* 500 - Internal Server Error


## Versions

* **絕對不要釋出一個沒有版號的API。**Never release an API without a version number.
* **版本必須是整數，不可為浮點數，並前以v為前綴字。**Versions should be integers, not decimal numbers, prefixed with ‘v’. For example: 
    * Good: v1, v2, v3
    * Bad: v-1.1, v1.2, 1.3
* **維護至少至前一個版本的API。** Maintain APIs at least one version back. 


## Record limits

* **如果沒有指定限制，則回傳預設的資料量限制。**If no limit is specified, return results with a default limit. 
* To get records 51 through 75 do this:
    * http://example.gov/magazines?limit=25&offset=50
    * offset=50 means, ‘skip the first 50 records’
    * limit=25 means, ‘return a maximum of 25 records’

**資料量限制和資料總數的資訊應該包含在回應裡。**
Information about record limits and total available count should also be included in the response. Example:

    {
        "metadata": {
            "resultset": {
                "count": 227,
                "offset": 25,
                "limit": 25
            }
        },
        "results": []
    }

## Request & Response Examples

### API Resources

  - [GET /magazines](#get-magazines)
  - [GET /magazines/[id]](#get-magazinesid)
  - [POST /magazines/[id]/articles](#post-magazinesidarticles)

### GET /magazines

Example: http://example.gov/api/v1/magazines.json

    {
        "metadata": {
            "resultset": {
                "count": 123,
                "offset": 0,
                "limit": 10
            }
        },
        "results": [
            {
                "id": "1234",
                "type": "magazine",
                "title": "Public Water Systems",
                "tags": [
                    {"id": "125", "name": "Environment"},
                    {"id": "834", "name": "Water Quality"}
                ],
                "created": "1231621302"
            },
            {
                "id": 2351,
                "type": "magazine",
                "title": "Public Schools",
                "tags": [
                    {"id": "125", "name": "Elementary"},
                    {"id": "834", "name": "Charter Schools"}
                ],
                "created": "126251302"
            }
            {
                "id": 2351,
                "type": "magazine",
                "title": "Public Schools",
                "tags": [
                    {"id": "125", "name": "Pre-school"},
                ],
                "created": "126251302"
            }
        ]
    }

### GET /magazines/[id]

Example: http://example.gov/api/v1/magazines/[id].json

    {
        "id": "1234",
        "type": "magazine",
        "title": "Public Water Systems",
        "tags": [
            {"id": "125", "name": "Environment"},
            {"id": "834", "name": "Water Quality"}
        ],
        "created": "1231621302"
    }



### POST /magazines/[id]/articles

Example: Create – POST  http://example.gov/api/v1/magazines/[id]/articles

    {
        "title": "Raising Revenue",
        "author_first_name": "Jane",
        "author_last_name": "Smith",
        "author_email": "jane.smith@example.gov",
        "year": "2012"
        "month": "August"
        "day": "18"
        "text": "Lorem ipsum dolor sit amet, consectetur adipiscing elit. Etiam eget ante ut augue scelerisque ornare. Aliquam tempus rhoncus quam vel luctus. Sed scelerisque fermentum fringilla. Suspendisse tincidunt nisl a metus feugiat vitae vestibulum enim vulputate. Quisque vehicula dictum elit, vitae cursus libero auctor sed. Vestibulum fermentum elementum nunc. Proin aliquam erat in turpis vehicula sit amet tristique lorem blandit. Nam augue est, bibendum et ultrices non, interdum in est. Quisque gravida orci lobortis... "

    }


## Mock Responses
**在測試的伺服器上建議每個資源都能接受一個模擬的參數。當傳遞這個參數時，應該回傳一個模擬的回應(略過後端)。**
It is suggested that each resource accept a 'mock' parameter on the testing server. Passing this parameter should return a mock data response (bypassing the backend).

**在開發初期實作這個功能能夠確保API表現一致的行為，並支援測試驅動開發(TDD)的開發方法。**
Implementing this feature early in development ensures that the API will exhibit consistent behavior, supporting a test driven development methodology.

**注意: 如果模擬參數包含在一個針對正式環境的請求中，應該要回傳錯誤。**
Note: If the mock parameter is included in a request to the production environment, an error should be raised.


## JSONP

JSONP is easiest explained with an example. Here's a one from [StackOverflow](http://stackoverflow.com/questions/2067472/what-is-jsonp-all-about?answertab=votes#tab-top):

> Say you're on domain abc.com, and you want to make a request to domain xyz.com. To do so, you need to cross domain boundaries, a no-no in most of browserland.

> The one item that bypasses this limitation is `<script>` tags. When you use a script tag, the domain limitation is ignored, but under normal circumstances, you can't really DO anything with the results, the script just gets evaluated.

> Enter JSONP. When you make your request to a server that is JSONP enabled, you pass a special parameter that tells the server a little bit about your page. That way, the server is able to nicely wrap up its response in a way that your page can handle.

> For example, say the server expects a parameter called "callback" to enable its JSONP capabilities. Then your request would look like:

>         http://www.xyz.com/sample.aspx?callback=mycallback

> Without JSONP, this might return some basic javascript object, like so:

>         { foo: 'bar' }

> However, with JSONP, when the server receives the "callback" parameter, it wraps up the result a little differently, returning something like this:

>         mycallback({ foo: 'bar' });

> As you can see, it will now invoke the method you specified. So, in your page, you define the callback function:

>         mycallback = function(data){
>             alert(data.foo);
>         };

http://stackoverflow.com/questions/2067472/what-is-jsonp-all-about?answertab=votes#tab-top

<a name="note-1"></a>註1:在URL加上版號的方法，有人支持也有人反對:    
    * http://stackoverflow.com/questions/389169    
    * http://stackoverflow.com/questions/972226    
    * [Web API Design](http://pages.apigee.com/web-api-design-ebook.html), by Brian Mulloy, Apigee
