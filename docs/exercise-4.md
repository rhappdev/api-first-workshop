# Introduction

In this exericse, we will explore the starter hello-world Node.js application built with [express](https://expressjs.com) and [oas-tools](https://www.npmjs.com/package/oas-tools) and perform following tasks:

* Include OpenAPI spec for the to-do list API.
* Implement endpoints for the API based on the specificaiton.
* Implement unit tests for the exposed endpoints.

# Prerequisites

[Nodejs - 8.0+](https://nodejs.org/en/download/)

# Part 1 -  Build ToDo List API by including OpenAPI specification

###### Step 1 - Explore the app

Git clone the [api-first-workshop-nodejs](https://github.com/rhappdev/api-first-workshop-nodejs) project to your local machine and install the dependencies:

```
git clone https://github.com/rhappdev/api-first-workshop-nodejs

cd api-first-workshop-nodejs

npm install

npm run dev
```

The app already contains the sample code for Hello World API based on OpenAPI specification located at `./src/definition/swagger.yaml`. You can execute a curl command `curl http://localhost:8001/api/hello` to verify that the server is working. You should receive an error response:
```
[{"message":"Missing parameter greeting in query. "}]
```

###### Step 2 - Include OpenAPI specification for the ToDo List API

* Download the specification for ToDo List API from Apicurio that we genereated in [Exercise 1](exercise-1.md). 
* Replace the contents of the file located at `./src/definition/swagger.yaml` with the ToDo List API specification. 

> Note: In case if you have not been able to finish [exercise-1](exercise-1.md), please download the complete ToDo List API specification from [here](../contract/swagger.yaml)

###### Step 3 - Verify the app is running without any errors

```
npm run dev
```

###### Step 4 - Interact with ToDo List API's resources
Review the ToDo List API's resources through Swagger UI located at `http://localhost:8001/docs`.

###### Step 5 - Define controllers for each operation (endpoints)
`oas-tools` looks for a special annotation in the `.yaml` file to map the endpoint to its Express router and controller when executing a request.

In `swagger.yaml`, define the controller for the operation `getItems` by adding a property `x-swagger-router-controller: itemRoute` just before `operationId: getItems`.

Likewise, please add the same property for remaining operations: `createItem`, `getItem`, `updateItem` and `deleteItem`.

###### Step 6 - Update `PUT` and `POST` operations with extra annotation

In `swagger.yaml`, add `x-name: item` property for `createItem` and `updateItem` operation just after the `requestBody` property, one level down (the indentation shold be the same as the property `description` inside `requestBody`).

 > OpenAPI Specification version 3 defines request's body in a different way. It is not a parameter as it was in Swagger version 2. Request bodies are defined in tne section `requestBody` which doesn't have property called `name`. It needs this property to work with the code that is genereated using `openapi-generator-cli` tool.

###### Step 7 - Run the tests

```
npm run test
```

Review the tests written in `/test/application.spec.ts`.

# Part 2 -  Implement routes and controllers for various operations (i.e. writing actual code)

###### Step 1 - Define controllers/functions for each operations declared in the OpenAPI specification.

* Create a directory `src/controllers` and a file `getItemsHander.ts` inside, and update it with following code:

  ``` javascript
    import { Request, Response } from "express";
    import * as P from "bluebird";
    import { TDebug } from "../log";
    import { ItemDAO } from "../dao/item.dao";

    const debug = new TDebug("nodejs:src:controllers:getItemsHandler");

    const itemDao = ItemDAO.getInstance();

    export async function getItemsHandler(req: Request, res: Response): P<any> {
      debug.log("getItemsHandler start");
      const items = itemDao.getItems();
      res.send(items);
      debug.log("getItemsHandler end");
    }
  ```

> **Note:** Data models and data access objects (DAO) are already defined as part of the base template.

* Implement the rest of the controllers for the operations/endpoints defined in the ToDo List OpenAPI spec. For reference, the solution has been provided in the branch `step2` of the [api-first-workshop-nodejs](https://github.com/rhappdev/api-first-workshop-nodejs/tree/step2) repository.


###### Step 2 - Define routes for the operations delcared in the OpenAPI specification (`swagger.yaml`)

* Create a directory `src/routes` and a new file `itemRoute.ts` inside it with the following content:

  * Include reference for `express` router and `asyncHandler`.
    ``` javascript
    import { Router } from "express";
    import { asyncHandler } from "../lib/asyncHandler";
    ```

  * Map the controllers with the corresponding route. Add the following lines for the `getItems` operation:

    ```javascript
    import { getItemsHandler } from "../controllers/getItemsHandler";

    export const getItems  = Router().use("/", asyncHandler(getItemsHandler, "getItems"));
    ```
  * Add the corresponding lines for all other operations according to the ToDo API specification. Refer to the implemented `itemRoute.ts` file in the `step2` branch of the [api-first-workshop-nodejs](https://github.com/rhappdev/api-first-workshop-nodejs/blob/step2/src/routes/itemRoute.ts) repository.

  * In the file `src/middlewares/swagger.ts` in the `initSwaggerMiddlware` change the option `router: false` to `router: true`. This enables the router middleware.

> **Note:** All the parameters and body objects are accessible through `req.swagger.params.xxx.value` attribute.

###### Step 3 - Verify the app is running fine

Start the app:
```
npm run dev
```

Go to http://localhost:8001/docs and make some API calls, you should receive  `200 OK` status code.


# Part 3 -  Implement unit tests for different routes

###### Step 1 - Define the tests
* Create a directory `test/routes` and a file `item.route.spec.ts` inside it.
* Update the file `item.route.spec.ts` with the following test case:

  ``` javascript
  import chaiHttp = require("chai-http");
  import app from "../../src/application";
  import { ItemDAO } from "../../src/dao/item.dao";
  import * as chai from "chai";

  const expect = chai.expect;
  chai.use(chaiHttp);
  describe("ItemRoute - Test ItemRoute endpoints", function () {
    it("getItems - Should retrieve and empty Array", (done: () => void): void => {
        chai.request(app)
            .get("/api/v1/items")
            .set("content-type", "application/json")
            .send({})
            .end((err: Error, res: any): void => {
                expect(res.statusCode).to.be.equal(200);
                expect(res.body.length).to.be.equal(0);
                done();
            });
    });
    it("getItems - Should retrieve and Item Array with one Item", (done: () => void): void => {
      const itemDAO = ItemDAO.getInstance();
      itemDAO.addItem({"description": "Description", name: "Name", id: "id"});
      chai.request(app)
          .get("/api/v1/items")
          .set("content-type", "application/json")
          .send({})
          .end((err: Error, res: any): void => {
              expect(res.statusCode).to.be.equal(200);
              expect(res.body.length).to.be.equal(1);
              expect(res.body[0].name).to.be.equal("Name");
              expect(res.body[0].description).to.be.equal("Description");
              itemDAO.deleteItem(res.body[0].id);
              done();
          });
    });
  });
  ```
###### Step 2 - Verify the test

```
npm run test
```

# Part 4 - Implement unit tests for data access objects 

###### Step 1 - Define the test
* Create a directory `test/dao` and a file `item.dao.spec.ts` inside it.
* Update the file `item.dao.spec.ts` with following test case:

  ``` javascript
  import { ItemDAO } from "../../src/dao/item.dao";
  import * as chai from "chai";
  import { Item } from "../../src/models/item";

  const expect = chai.expect;
  describe("ItemDAO - Test ItemDAO Methods", function () {
    const itemDAO = ItemDAO.getInstance();
    it("getItems - Should retrieve an empty array", (done: () => void): void => {
      expect(itemDAO.getItems().length).to.be.equal(0);
      done();
    });
    it("addItem - Should add an item", (done: () => void): void => {
        itemDAO.addItem({"description": "Description", name: "Name", id: "id"});
        expect(itemDAO.getItems().length).to.be.equal(1);
        expect(itemDAO.getItems()[0].description).to.be.equal("Description");
        expect(itemDAO.getItems()[0].name).to.be.equal("Name");
        done();
    });
    it("getItems - Should retrieve an item", (done: () => void): void => {
      expect(itemDAO.getItems().length).to.be.equal(1);
      done();
    });
    it("getItem - Should retrieve an item", (done: () => void): void => {
      const item = itemDAO.getItems()[0];
      expect(itemDAO.getItem(item.id)).not.to.be.equal(undefined);
      done();
    });
    it("getItem - Should not retrieve an item", (done: () => void): void => {
      expect(itemDAO.getItem("xxxx-ssssss-sssss")).to.be.equal(undefined);
      done();
    });
    it("updateItem - Should update an item", (done: () => void): void => {
      const item = itemDAO.getItems()[0];
      item.description = "Description 2";
      item.name = "Name 2";
      itemDAO.updateItem(item);
      expect(itemDAO.getItems()[0].name).to.be.equal("Name 2");
      expect(itemDAO.getItems()[0].description).to.be.equal("Description 2");
      done();
    });
    it("deleteItem - Should delete an item", (done: () => void): void => {
      const item = itemDAO.getItems()[0];
      itemDAO.deleteItem(item.id);
      expect(itemDAO.getItems().length).to.be.equal(0);
      done();
    });
  });
  ```

###### Step 2 - Verify the test

```
npm run test
```

# Part 5 - Linking the front-end app and the back end implementation

* Start the API back end:

  ```
  npm run dev
  ```

* In the local copy of the `api-first-workshop-angular` application, in `src/app/environments/environment.ts` change the `API_URL` to `http://localhost:8001/api/v1`.

* Start the Angular app
  ```
  ionic serve
  ```
and go to http://localhost:8100

Verify that the application works as expected.
