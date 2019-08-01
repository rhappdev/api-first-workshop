# Introduction

In this exercise we will learn how we can create and test the API contract using [Postman](https://www.getpostman.com) and [Newman](https://github.com/postmanlabs/newman). This exercise will step you through how to create a mock server and run integration tests against the various endpoints.

## Step 1 - Locate completed contract

For this exercise, you will require a complete and valid OpenAPI spec. If you have completed the steps in [exercise-1](/docs/exercise-1.md), continue on to the next step.

If not, you can download the completed version of the the contract [here](https://github.com/primashah/workshop/blob/master/contract/swagger.yaml). Be sure to download the file to the directory you are working in.

## Step 2 - Postman, OpenAPI and a Contract

Now that you have the complete OpenAPI spec, open Postman, in the top left corner of the application window, click **Import** button to reveal the file explorer.

![File Importer](/docs//images/exercise-2/postman-file-importer.png)

Click **Choose Files** and navigate the to the directory were your OpenAPI file is saved.

If the OpenAPI file is valid and can be parsed, a new collection with the label _ToDo List_ will appear in Postman's collections sidebar.

## Step 3 - Exploring our imported API Spec

In Postman's collection explorer, expand the _ToDo List_ collection and any subdirectories and examine the contents.

![Collection Explorer](/docs//images/exercise-2/postman-imported-coll.png)

You should find 5 endpoints, defined previously in Apicurio studio. Let's explore one of these items in more detail.

Select the item with label _List All Items_. This is a `GET` HTTP operation that will return an array of to-do items.

In the top right corner, we can see examples of our response. Click _Examples(2)_ and select an item from the list to reveal example responses.

![Example Response](/docs//images/exercise-2/postman-response-example.png)

## Step 4 - Setting up our Mock Server

We will set up a server to run our tests. Luckily, Postman offers us a [mock server](https://learning.getpostman.com/docs/postman/mock_servers/setting_up_mock/) out of the box. This provides the ability to run the tests and validate the API schema. Another advantage is that we can run a mock server and allow the developers to retrieve mocked responses until the back end is implemented.

The responses that the mock server will return are based on the Examples configured for the endpoints in the collection.

Let's set up a mock server for our To-Do collection in Postman.

1. Select the parent row of the _ToDo List_ collection from the collection explorer to the left of the window.
   ![Mock Server - 1](/docs//images/exercise-2/mock-server-1.png)
2. Select the right facing arrow to expand collection settings. Click the **...** button and select _Mock Collection_ from the options.
   ![Mock Server - 2](/docs//images/exercise-2/mock-server-2.png)
3. The next window will present you with options to configure the server, enter the title and click the **Create** button to create a mock server.
   ![Mock Server - 3](/docs//images/exercise-2/mock-server-3.png)
4. The next screen will show the URL of the created mock server. Click on the link and copy the URL from the URL field. You can also get the link later by clicking again on the right arrow and going to _Mocks_ tab. The link will look like: `https://66e2a15a-f50e-4292-b753-91084abf70c0.mock.pstmn.io`.

## Step 5 - Testing, testing testing!

Now that we have explored Postman and set up our mock server, we will add some basic tests to validate our API contract.

But first we'll need to configure an [Environment](https://learning.getpostman.com/docs/postman/environments_and_globals/manage_environments/) for the tests and set the variable `{{"{{ baseUrl }}"}}` that is created automatically for each endpoint on import to the value of our mock server.

**Note:** in some versions of Postman this environment is created automatically on creating a mock server.

* Click on the gear icon in the top-right corner to open the Manage Environments pop-up.
* Click **Add**
* Enter a name for the new environment, e.g. _ToDo List_
* Add a variable `baseUrl` with the Initial Value set to the mock server URL (e.g. `https://66e2a15a-f50e-4292-b753-91084abf70c0.mock.pstmn.io`).
* Click **Add**, confirm that the environment has been created, and close the window.
* From the drop-down list (that shows `No Environment` or the name of another environment) select your newly created _ToDo List_ environment.

#### GET - List All Items

First, we will update the success response of the _List All Items_ endpoint. 

1. Navigate to the _Examples_ and select the _Successful Response_ to reveal the editor.

2. Copy and paste the array of to-do items below, into the example response body editor. Click **Save Example** and return to request window.

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

3. Select the _Tests_ tab and paste the sample code below:
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

4. Verify the tests are running by clicking the **Send** button.

Note that as we have 2 examples: one with successful `200` status code, and another one with `500` error code, the mock server may send the error one.
You can add `x-mock-response-code` header to the request with the value of the status code of the example you want to be returned. You can add it in the _Headers_ tab of the request. Alternatively, you can delete the error response example.
Read more about how to mock with examples in [Postman documentation](https://learning.getpostman.com/docs/postman/mock_servers/mocking_with_examples/).

#### POST - Create an Item

Next we want to confirm we have created a to-do item successfully. We do this by testing the status code returned is as decribed in the schema. First, we will update the example success response.

1. In the example request body, paste the object below. This will be our POST body. Save and return to the request tab.

```json
{
  "id": "1234",
  "name": "New Item",
  "description": "New item spiel"
}
```

2. Next add a test to check the response statusCode is as expected. Copy & paste snippet below into the test tab.

```javascript
pm.test("Return a successful status code", function() {
  pm.expect(pm.response.code).to.be.oneOf([200, 201]);
});
```

3. Verify tests by selecting the **Send** button.

#### GET - Get an Item

For this GET request, an `itemId` is passed as a parameter and a single object is returned. We will use our tests to validate the schema of the response and the expected status code. As with the previous intrustions, navigate to the `Examples` tab for this request and open the editor.

1.  In the example request URL/address bar, replace the `<string>` with `1234`.
2.  In the example request body, paste the object below, click **Save Example** and close the window to go back to the request tab.

```json
{
  "id": "1234",
  "name": "Test",
  "description": "Test item 1"
}
```

4. In the _Params > Path Variables_ set the value of `itemId` to `1234`.
5. Now add the test below to the _Tests_ tab. Save changes.

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

6. Verify tests by clicking the **Send** button.

#### PUT - Update an Item

1. In the example request URL/address bar, replace the `<string>` with `1234`.
2. In the example request body, paste the object below, click **Save Example** and close the window to go back to the request tab.
3. In the _Params > Path Variables_ set the value of `itemId` to `1234`.
4. In the request body tab paste the object below.

```json
{
  "id": "1234",
  "name": "New Test",
  "description": "New test description"
}
```

4.  Now add the test below to the tests tab. Save changes.

```javascript
pm.test("Returns status code of 202", function() {
  pm.response.to.have.status(202);
});
```

5. Verify tests by selecting the **Send** button.

#### DELETE - Delete Item

1. In the example request URL/address bar, replace the `<string>` with `1234`.
2. Change the Status to `204 No Content`.
3. Click **Save Example** and close the window to go back to the request tab.
4. In the _Params > Path Variables_ set the value of `itemId` to `1234`.

```javascript
pm.test("Returns status code of 204", function() {
  pm.response.to.have.status(204);
});
```

5. Verify tests by selecting the **Send** button.

## Step 6 - The CLI, Newman and our Tests

OK, so up to now we have manually verified our test against the schema by clicking the **Send** button for each request. In this section we are going to export our Postman collection and run our tests in the command line using a tool called [Newman](https://learning.getpostman.com/docs/postman/collection_runs/command_line_integration_with_newman/).

#### Exporting the Postman collections

1. From the collection explorer, expand the collection settings pane.
2. Click the **...** button to reveal the hidden menu.
3. Select _Export_ from the options.
4. In the next window, select the 3rd option, _Collection v2.1 (recommended)_ and click **Export**.
5. Add a new directory to the root of your working directory, named `postman`.
6. Export collection to the the `postman` directory, change the file name to `collection.json`.

#### Exporting the Postman environment

1. Click the settings "gear" icon in the top right corner.
2. Find the _ToDo List_ environment config and click the _Download Environment_ (arrow pointing down) icon.
3. Export the environment config to the the `postman` directory, change the filename to `environment.json`.

#### Installing Newman

1. Open a command line application of your choice.
2. Navigate to the working directory.
3. Install `newman` via npm.

```bash
npm i -g newman
```

4. In the command line enter the following command to run the tests:

```bash
newman run ./postman/collection.json -e ./postman/environment.json
```
