# **API Introduction**

The **GET** request method is used by a client to fetch data from a server. If the requested resource is found on the server, it will then be sent back to the client.

The **PUT** method replaces existing data or creates data if it does not exist. If you use PUT many times, it will have no effect — there will only be one copy of the dataset on the server.

The **POST** method is used primarily to create new resources. Using POST many times will add data in multiple places on the server. **It is recommended to use PUT to update resources and POST to create new resources**.

The **DELETE** method will remove data or resources specified by the client on a server.

**Endpoints** are access points to data or computing resources hosted on a server and they take the form of an [HTTP URI](https://en.wikipedia.org/wiki/Uniform_Resource_Identifier). Endpoints are added to an API's base URL (e.g. `http://example.com`) to create a path to a specific resource or container of resources. The following are some examples of endpoints:

*...collections of individually-addressable resources... The resources and methods are known as nouns and verbs of APIs. With the HTTP protocol, **the resource names naturally map to URLs, and methods naturally map to HTTP methods**...*

APIs that utilize the HTTP protocol, request methods, and endpoints are referred to as **RESTful APIs**. REST (Representational State Transfer) is an architectural style that prescribes standards for web-based communication. The Google [description of a RESTful system](https://developers.google.com/photos/library/guides/about-restful-apis):

Besides query strings, RESTful APIs can also use the following fields in their requests:

- **Headers**: parameters that detail the HTTP request itself.
- **Body**: data that a client wants to send to a server.

The body is written in the **JSON** or **XML** data formatting language.

JSON supports the following data types:

- **Numbers**: all types — no distinction between integers and floating point values.
- **Strings**: text enclosed in quotes.
- **Booleans**: True or False values.
- **Arrays**: a list of elements grouped by similar type.
- **Null**: an "empty" value.

**Authentication** refers to the process of determining a client's identity.

**Authorization** refers to the process of determining what permissions an authenticated client has for a set of resources.

## **Bucket Create**

1. 만들 버킷정보가 담긴 json 파일 작성

        nano values.json​#### Content ####{  "name": "<YOUR_BUCKET_NAME>",   "location": "us",   "storageClass": "multi_regional"}

2. 버킷 권한 설정 및 버킷 생성
    - [OAuth 2.0 Playground](https://developers.google.com/oauthplayground/) 접속 후 빨간 박스선택 후 Authorize APIs 클릭

        ![](https://cdn.qwiklabs.com/DQ0hlDDMV5APsKqVuQKBklSVrkMQ6KIHmYmlHeAxOWE%3D)

    - Exchange authorization code for tokens 클릭 후 Access token 복사

        ![](https://cdn.qwiklabs.com/qR7J7L1r4Qc0N038G4COgd5sJAbMeuxbf5YfBZPjXVQ%3D)

3. 환경변수 설정 및 request

        # 환경변수 설정export OAUTH2_TOKEN=<YOUR_TOKEN>export PROJECT_ID=<YOUR_PROJECT_ID>​# 버킷생성 요청curl -X POST --data-binary @values.json \    -H "Authorization: Bearer $OAUTH2_TOKEN" \    -H "Content-Type: application/json" \    "https://www.googleapis.com/storage/v1/b?project=$PROJECT_ID"    # 버킷네임은 유일해야한다!    # 버킷에 이미지 저장curl -X POST --data-binary @$OBJECT \    -H "Authorization: Bearer $OAUTH2_TOKEN" \    -H "Content-Type: image/png" \    "https://www.googleapis.com/upload/storage/v1/b/$BUCKET_NAME/o?uploadType=media&name=demo-image"​