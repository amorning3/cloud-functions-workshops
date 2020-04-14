<!--
#
# Licensed to the Apache Software Foundation (ASF) under one or more
# contributor license agreements.  See the NOTICE file distributed with
# this work for additional information regarding copyright ownership.
# The ASF licenses this file to You under the Apache License, Version 2.0
# (the "License"); you may not use this file except in compliance with
# the License.  You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#
-->

#  API Management of web actions

Let's now explore how these web actions can be turned into an API using the [API Gateway](https://cloud.ibm.com/docs/openwhisk?topic=cloud-functions-apigateway). 

First, we will choose `/myapi` as the base path. This is the part of the path before the actual endpoint. For example: `example.com/basepath/endpoint`. This is useful for grouping endpoints together in a logical way and is how IBM Cloud Functions (ICF) organizes your endpoints into a single API.

At first glance, this does not appear to be different from web actions -- you tell ICF to make your action or sequence to be invoked by calls to an external HTTP endpoint. It returns a URL for an HTTP endpoint. However, the key difference is that you are going through the API Gateway and are able to take advantage of a variety of features outside the scope of this course including [rate limiting, authentication](https://cloud.ibm.com/docs/api-gateway?topic=api-gateway-create_api), and [custom domains](https://cloud.ibm.com/docs/api-gateway?topic=api-gateway-custom_endpoint) for your API. This is all done by the API Gateway with no changes to your action, whereas it would have to be handled by your action directly or just simply impossible to do if you were to just use web actions.

## Create an API

To create an API, run the following command syntax:

```bash
ibmcloud fn api create BASE_PATH API_PATH API_VERB ACTION
```

{% hint style="warning" %}
Be advised that the example above is merely an example of the syntax used to create an API endpoint and should **not** be run.
{% endhint %}

### Hello endpoint

First you need to create the `hello` endpoint. Note that all actions used in an API must be web actions, so you mustn't forget to run `ibmcloud fn action update hello --web true` prior to running the commands below.

1. Run the following command to create a simple GET endpoint for the `hello` action:

    ```bash
    ibmcloud fn api create /myapi /foo get hello
    ```

    ```bash
    ok: created API /myapi/foo GET for action /_/hello
    https://service.us.apiconnect.ibmcloud.com/gws/apigateway/api/aac6bc4a6f94f19dd008e64513b62bf155af5703a95a142f0c9a6ea83af81300/myapi/foo
    ```

2. Check to see that the API was created:

    ```bash
    ibmcloud fn api list
    ```

    ```bash
    ok: APIs
    Action                                     Verb  API Name  URL
    /joesphine.watson@gmail.com_ns/hello        get    /myapi  https://service.us.apiconnect.ibmcloud.com/gws/apigateway/api/d9903f40439f1a268b7dcbac42a389cdde605f3f3bef57f69789be6df438361e/myapi/hello
    ```

3. Now let’s invoke that API via curl:

    ```bash
    curl https://service.us.apiconnect.ibmcloud.com/gws/apigateway/api/aac6bc4a6f94f19dd008e64513b62bf155af5703a95a142f0c9a6ea83af81300/myapi/foo
    ```

    ```json
    {
    "greeting": "Hello, undefined from undefined"
    }
    ```
    
You have now created and invoked your first IBM Cloud Functions (ICF) API endpoint!

### Other response types

You must be remember that, by default, ICF expects a functions return `content-type` will be JSON. Since for the remaining functions this is not the case, you must be mindful to set the appropriate response types with the `--response-type <TYPE>` flag. Valid type values include `http`, `json`, `text`, and `svg`. Let's demonstrate how to create non-JSON endpoints by creating an endpoint for your redirect web action.

1. Create the endpoint for the HTTP endpoint:

    ```bash
    ibmcloud fn api create /myapi /redirect get redirect --response-type http
    ```

    ```bash
    ok: created API /myapi/redirect GET for action /_/redirect
    https://service.us.apiconnect.ibmcloud.com/gws/apigateway/api/d9903f40439f1a268b7dcbac42a389cdde605f3f3bef57f69789be6df438361e/myapi/redirect
    ```

2. Check to make sure that the redirect works by copying and pasting the URL returned into your browser.

You’ve now successfully created an endpoint for an HTTP response type. The other response types listed above follow the same syntax. For the sake of brevity, you will not be shown the creation of the other response types or web actions created in the previous section. However, if you would like to go through the exercise of creating those API endpoints, you can walk through the optional exercise [Other APIs](other_apis.md).

## Use OpenAPI Specification

As you can begin to tell, as the number of API endpoints increase, documenting and managing them becomes increasingly difficult. One solution to this is to use the [OpenAPI Specification](https://swagger.io/specification/). This has a plethora of tools around for documenting, creating stub projects, and more, in a variety of languages. And it is supported by ICF!

1. Let's take stock of our API by listing out all the endpoints:

    ```bash
    ibmcloud fn api list
    ```

    ```bash
    ok: APIs
    Action                                     Verb  API Name  URL
    /josephine.watson@gmail.com_ns/hello        get    /myapi  https://service.us.apiconnect.ibmcloud.com/gws/apigateway/api/d9903f40439f1a268b7dcbac42a389cdde605f3f3bef57f69789be6df438361e/myapi/hello
    /josephine.watson@gmail.com_ns/html         get    /myapi  https://service.us.apiconnect.ibmcloud.com/gws/apigateway/api/d9903f40439f1a268b7dcbac42a389cdde605f3f3bef57f69789be6df438361e/myapi/html
    /josephine.watson@gmail.com_ns/manual       put    /myapi  https://service.us.apiconnect.ibmcloud.com/gws/apigateway/api/d9903f40439f1a268b7dcbac42a389cdde605f3f3bef57f69789be6df438361e/myapi/manual
    /josephine.watson@gmail.com_ns/redirect     get    /myapi  https://service.us.apiconnect.ibmcloud.com/gws/apigateway/api/d9903f40439f1a268b7dcbac42a389cdde605f3f3bef57f69789be6df438361e/myapi/redirect
    /josephine.watson@gmail.com_ns/atom         get    /myapi  https://service.us.apiconnect.ibmcloud.com/gws/apigateway/api/d9903f40439f1a268b7dcbac42a389cdde605f3f3bef57f69789be6df438361e/myapi/atom
    ```
{% hint style="warning" %}
If you did not do the optional exercise, your output will be smaller as you have fewer endpoints. That is fine for the purposes of this demonstration.
{% endhint %}

2. View the OpenAPI Specification for `myapi` using the following command:

    ```bash
    ibmcloud fn api get /myapi
    ```

    You should see a long JSON document that starts something like:

    ```json
    {
        "swagger": "2.0",
        "basePath": "/myapi",
        "info": {
            "title": "/myapi",
            "version": "1.0.0"
        },
        ...
    }
    ```

3. You will want to write that to a file with:

    ```bash
    ibmcloud fn api get /myapi > myapi.json
    ```

    Now you can delete the existing API you created and restore using this document.

4. Delete the existing API:

    ```bash
    ibmcloud fn api delete /myapi
    ```
    ```bash
    ok: deleted API /myapi
    ```

5. Check that the endpoints are gone:

    ```bash
    ibmcloud fn api list
    ```
    ```bash
    ok: APIs
    Action                            Verb             API Name  URL
    ```

6. Restore the endpoints from the OpenAPI Specification:

    ```bash
    ibmcloud fn api create -c myapi.json
    ```

    ```bash
    ok: created API /myapi/redirect get for action /josephine.watson@gmail.com_ns/redirect
    https://service.us.apiconnect.ibmcloud.com/gws/apigateway/api/d9903f40439f1a268b7dcbac42a389cdde605f3f3bef57f69789be6df438361e/myapi/redirect
    ok: created API /myapi/hello get for action /josephine.watson@gmail.com_ns/hello
    https://service.us.apiconnect.ibmcloud.com/gws/apigateway/api/d9903f40439f1a268b7dcbac42a389cdde605f3f3bef57f69789be6df438361e/myapi/hello
    ok: created API /myapi/html get for action /josephine.watson@gmail.com_ns/html
    https://service.us.apiconnect.ibmcloud.com/gws/apigateway/api/d9903f40439f1a268b7dcbac42a389cdde605f3f3bef57f69789be6df438361e/myapi/html
    ok: created API /myapi/manual put for action /josephine.watson@gmail.com_ns/manual
    https://service.us.apiconnect.ibmcloud.com/gws/apigateway/api/d9903f40439f1a268b7dcbac42a389cdde605f3f3bef57f69789be6df438361e/myapi/manual
    ok: created API /myapi/atom get for action /josephine.watson@gmail.com_ns/atom
    https://service.us.apiconnect.ibmcloud.com/gws/apigateway/api/d9903f40439f1a268b7dcbac42a389cdde605f3f3bef57f69789be6df438361e/myapi/atom
    ```

7. You can now see that endpoints are restored:

    ```bash
    ibmcloud fn api list
    ```

    ```bash
    ok: APIs
    Action                                     Verb  API Name  URL
    /josephine.watson@gmail.com_ns/atom         get    /myapi  https://service.us.apiconnect.ibmcloud.com/gws/apigateway/api/d9903f40439f1a268b7dcbac42a389cdde605f3f3bef57f69789be6df438361e/myapi/atom
    /josephine.watson@gmail.com_ns/hello        get    /myapi  https://service.us.apiconnect.ibmcloud.com/gws/apigateway/api/d9903f40439f1a268b7dcbac42a389cdde605f3f3bef57f69789be6df438361e/myapi/hello
    /josephine.watson@gmail.com_ns/html         get    /myapi  https://service.us.apiconnect.ibmcloud.com/gws/apigateway/api/d9903f40439f1a268b7dcbac42a389cdde605f3f3bef57f69789be6df438361e/myapi/html
    /josephine.watson@gmail.com_ns/manual       put    /myapi  https://service.us.apiconnect.ibmcloud.com/gws/apigateway/api/d9903f40439f1a268b7dcbac42a389cdde605f3f3bef57f69789be6df438361e/myapi/manual
    /josephine.watson@gmail.com_ns/redirect     get    /myapi  https://service.us.apiconnect.ibmcloud.com/gws/apigateway/api/d9903f40439f1a268b7dcbac42a389cdde605f3f3bef57f69789be6df438361e/myapi/redirect
    ```

This OpenAPI Specification can now be stored in your code repository and used to update endpoints, documentation, or event generate stub code!

{% hint style="success" %}
Congratulations on successfully creating serverless APIs! Exposing and managing serverless APIs takes minimal effort using ICF and allows you full access control and use of the OpenAPI specification!
{% endhint %}
