# Introduction

In this exericse , we will explore the starter hello-world nodejs application built with express and [oas-tools](https://www.npmjs.com/package/oas-tools) and perform following tasks:


* Include openAPI spec for todolist API
* Implement endpoints for the API based on the specificaiton
* Implement unit tests for the exposed endpoint.

# Pre-reqs

[Nodejs - 8.0+](https://nodejs.org/en/download/)

# Part 1 -  Build Todolist API by including OpenAPI specification

### Step 1 - Explore the app

 Git clone the api-first-workshop-nodejs project to your local machine and checkout the master branch using the following.

```
git clone https://github.com/rhappdev/api-first-workshop-nodejs

cd api-first-workshop-nodejs

npm install

npm run dev

```


> Note: The app already contains the sample code for hello-world api based on hello-world openAPI specification.

### Step 2 - Include openAPI specification for todolist API.

* Download the specification for todolist API from APICURIO that we genereated in previous exercise 
* Replace it with  swagger.yaml file located at ./src/definition/ 

> Note: In case if you have not been able to finish, please download the complete todolist API specification from [here](../contract/swagger.yaml)


### Step 3 - Verify app is running without any errors.

```
npm run dev
```
### Step 4 - Interact with todolist API's resources
Review the todolist API's resources through swagger-ui located at `http://localhost:8001/docs`.

### Step 5 - Define handlers/controllers for the endpoints/operations.  
oas-tools looks for a special annotation in .yaml file to link the endpoint to its express router and controller when a excuting a request.

  * In swagger.yaml, define the controller for operation "/getitems" :

    * Just before `operationId: getItems`, add property `x-swagger-router-controller: itemRoute`

  
> Likewise, please add the same property for remaining operations: createItem, getItem, updateItem and deleteItem.


## Step X : todo: @Mikel do we need to include step x-name item? is it not something we will define in apicurio ?

### Step 6 - Verify app is running ok.

```
npm run test
```

> Review the test written in application.spec.ts located at '/test/applicaiton.spec.ts'. 


# Part 2 -  Implement routes and hanlders for various operations (i.e. writing actual code)

### Step 1 - Define routes for the operations delcared in the openAPI specificaiton (`swagger.yaml`).
  * Create a new file `itemRoute.ts` at `src/routes`.
  * Include reference for express router and asyncHandler.
  ```javasript
  import { Router } from "express";
  import { asyncHandler } from "../lib/asyncHandler";
  ```

### Step 2 - Define controllers/functions for each operations declared in the openAPI specification.

* Create file getItemsHander.ts at `src\controllers` and update it with following code.
 
    ```javasript
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

> Note: Data models and data access objects (Dao) are already defined as part of base template.


* Update the `itemRoute.ts` to map the controller with the corresponding route.

 ```javascript
  export const getItems  = Router().use("/", asyncHandler(getItemsHandler, "getItems"));
 ```
 
> Note: All the parameters and body objects are accessible through  req.swagger.params.xxx.value attribute.


### Step 3 - Verify app is running fine.
You should able to retrive list of items with 200 Http status code.
```
npm run dev
```
 
> Likewise implement the rest of the operations\endpoints defined in the todlist openAPI spec. For reference, the solution has been provided in branch named "step2".



# Part 3 -  Implement unit tests for different routes.

### Step 1 - Define the tests .
* Create file `item.route.spec.ts` in `test/routes` folder.
* Update file `item.route.spec.ts` with following test case:
```javascript
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
### Step 2 - Verify the test .
```
npm run test
```

# Part 3 -  Implement unit tests for Data access objects 
### Step 1 - Define the test.
* Create file `item.dao.spec.ts` in `test/dao` folder.
* Update file `item.dao.spec.ts` with following test case:

```javascript
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

#### Step 2 - Verify the test .
```
npm run test
```






   











