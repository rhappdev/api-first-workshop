# Introduction

In this exercise we will learn how we can create and test the API contract using [Postman]() and [Newman](). This exercise will step you through how to create a mock server and run pur integeration tests against the various endpoints.

## Step 1 - Locate completed contract

For this exercise, you will require a complete and valid swagger doc. If you have completed the steps in [exercise-1](/docs/exercise-1.md) continue on to the next step.

If not you can download a completed version of the the contract [here.](https://github.com/primashah/workshop/blob/master/contract/swagger.yaml) Be sure to download the file to the directory you are working in.

## Step 2 - Postman, OpenAPI and a Contract

Now that you have the complete swagger doc, open Postman, in the top left hand corner of the application window, select _**Import**_ button to reveal the file explorer.

![File Importer](/docs//images/exercise-2/postman-file-importer.png)

Select choose files and navaigate the to the directory were you swagger file is save.

If the swagger file is vaild and can be parsed, a new collection with the label `ToDo List` will appear in Postman's collections sidebar.

## Step 3 - Exploring our imported API Spec

In Postmans collection explorer, collapse the ToDo List collection and any subdirectories and examine the contents.

![Collection Explorer](/docs//images/exercise-2/postman-imported-coll.png)

You should find 5 items, describing them ake of each http to that assocted endpoint, anging from get an item to delete and item. Lets explore one of these items in more detail.

Select the item with label `List All Items`. This is a GET HTTP Operation that will return all valid Todo items in a array.

To the right hand corner, we can see examples of our response. Click `Examples` and select an item from the list to reveal example responses.

![Example Response](/docs//images/exercise-2/postman-response-example.png)

## Step 4 - Setting up our Mock Server

We will to setup a server to run our tests. Luckily, Postman offer us a [mock server](https://learning.getpostman.com/docs/postman/mock_servers/setting_up_mock/) out of the box. This provides the ability to run the tests and validate the api schema.

Let's setup a mock server for our Todo collection in Postman.

1. Select the parent row of the ToDo List collection from the collection explorer to the left of the window.
   ![Mock Server - 1](/docs//images/exercise-2/mock-server-1.png)
2. Select the right facing arrow to expand collection settings. Navigate to the `more button` and select `Mock Collection` from the options.
   ![Mock Server - 2](/docs//images/exercise-2/mock-server-2.png)
3. The next window will present you with options to configure the server, select the `Create` button to create mock server.
   ![Mock Server - 3](/docs//images/exercise-2/mock-server-3.png)

## Step 5 - Testing, testing testing!

Now that we have explored POstmanand setup our mock server. Next we will adds ome basic test to validate our api contract.

#### GET - List All

First up, the List All endpoint. First, we will update the success response. As described ealrier, navigate to the `Examples` and select the Succsss Response to reveal editor.

1. In the example request url/address bar, replace the `{{ baseUrl }}` with `{{ url }}`.
2. Copy and paste the array of todo items below, into the Example response body editor. Save and return to request window.

```json
[
  {
    "id": "1234",
    "name": "Test",
    "description": "First test item"
  },
  {
    "id": "1235",
    "name": "test 2",
    "description": "Second test item"
  }
]
```

3. Select the test tab and paste the sample code below.
4. In the request url/address bar, replace the `{{ baseUrl }}` with `{{ url }}`.

```javascript
var body = JSON.parse(responseBody);
var schema = {
  todos: {
    type: "array"
  }
};

pm.test("Status code is 200", function() {
  pm.response.to.have.status(200);
});

pm.test("Response body matches expected schema", function() {
  pm.expect(tv4.validate(body, schema)).to.be.true;
});
```

5. Verify the tests is running by selecting the `Send` button.

#### POST - Create Todo Item

Next we want to confirm we have created a todo item successfully. We do this by testing the status code retunred is that as decribed in the schema. First, we wil l update the example success response.

1. In the example request url/address bar, replace the `{{ baseUrl }}` with `{{ url }}`.
2. In the example request body, paste the object below. This will be our post body. Save and return to request tab

```json
{
  "id": "1234",
  "name": "New Item",
  "description": "New item spiel"
}
```

3. Next add a test to check the response statusCode is as expected. Copy & paste snippet below into the test tab.

```javascript
pm.test("Return a successful status code", function() {
  pm.expect(pm.response.code).to.be.oneOf([200, 201]);
});
```

4. In the request url/address bar, replace the `{{ baseUrl }}` with `{{ url }}`.
5. Verify tests by selecting the `Send` button.

#### GET - Get Item

For this GET request, an `itemId` is passed as a param and a single object is returned. We will use our tests to validate the schema of the response and the expected statusCode. As with the previous intrustions, navigate tot he `Examples` tab for this request and open the editor.

1.  In the example request url/address bar, replace the `{{ baseUrl }}` with `{{ url }}`.
2.  In the example request url/address bar, replace the `<string>` with `1234`.
3.  In the example request body, paste the object below, save chnages and close window.

```json
{
  "id": "1234",
  "name": "Test",
  "description": "Test item 1"
}
```

4. In the `params` set the value of `itemId` to `1234`.
5. Now add the test below to the tests tab. Save changes.

```javascript
var body = JSON.parse(responseBody);
var schema = {
  properties: {
    id: {
      type: "string"
    },
    name: {
      type: "string"
    },
    description: {
      type: "string"
    }
  }
};

pm.test("Status code is 200", function() {
  pm.response.to.have.status(200);
});

pm.test("Response body matches expected schema", function() {
  pm.expect(tv4.validate(body, schema)).to.be.true;
});
```

6. In the request url/address bar, replace the `{{ baseUrl }}` with `{{ url }}`.
7. Verify tests by selecting the `Send` button.

#### PUT - Update Item

1. In the example request url/address bar, replace the `{{ baseUrl }}` with `{{ url }}`.
2. In the example request body, paste the sample request object below, save changes and close window.
3. In the `params` set the value of `itemId` to `1234`.
4. In the request body tab paste th eobject below.

```json
{
  "id": "1234",
  "name": "New Test",
  "description": "New test description"
}
```

5.  Now add the test below to the tests tab. Save changes.

```javascript
pm.test("Returns status code of 202", function() {
  pm.response.to.have.status(202);
});
```

6. In the request url/address bar, replace the `{{ baseUrl }}` with `{{ url }}`.
7. Verify tests by selecting the `Send` button.

#### DELETE - Delete Item

1. In the example request url/address bar, replace the `{{ baseUrl }}` with `{{ url }}`.
2. In the example request url/address bar, replace the `:itemId` to `1234`.
3. Save a close editor, head back to request tab.
4. In the `params` set the value of `itemId` to `1234`.

```javascript
pm.test("Returns status code of 204", function() {
  pm.response.to.have.status(204);
});
```

5. In the request url/address bar, replace the `{{ baseUrl }}` with `{{ url }}`.
6. Verify tests by selecting the `Send` button.

## Step 6 - The CLI, Newman and our Tests

Ok so up to now we have manually verified our test against the schema by selecting the `Send` button for each request. In this section we are going to export our postman collection and run our tests in the command line using a tool called [Newman](https://learning.getpostman.com/docs/postman/collection_runs/command_line_integration_with_newman/).

#### Exporting the Postman collections

1. From the collection explorer, expand the collection settings pane.
2. Select the `more` butotn to reveal hidden menu.
3. Select `Export` from the options.
4. In the next window, select the 3rd option, `Collection v2.1`.
5. Add a new directory to the root of your working directory, named `postman`
6. Export collection to the the `postman` directory.

#### Exporting the Postman environment

1. Select the `Settings` icons from the top right.
2. Find the `ToDo List` environment config and sleect the export icon.
3. Export environment config to the the `postman` directory.

#### Installing Newman

1. Open a command line application of your choice.
2. Navigate to the working diretory
3. Install newman via npm

```bash
npm i -g newman
```

4. In the command line enter te folowiing command to run the tests.

```bash
newman run ./postman/collection.json -e ./postman/environment.json
```
