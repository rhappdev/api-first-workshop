# Guide

Currently we have an starter NodeJS project, usin express and oas-tools.

## Useful commands

`npm run build:live` : Generates the ts files to js files, in dist folders. Also checks tslint.

`npm run test` : fires unit tests.

`npm run dev` : Runs the app.

Note: More information avaliable the projects README.md


# Workshop step1

Currently the template it's using the default helloworld specification file. Follow the following instructions to update the NodeJS service with our ToDo List API.

1. Download from API Curio the ToDo List definition file that we have generated. If you have not been able to finish it use the following file:
   [swagger.yaml](../contract/swagger.yaml)
2. Replace the `swagger.yaml` file located at `src/definition/swagger.yaml`
3. run `npm run dev`, you will see how the service is started
4. Check the swagger-ui created doc at `http://localhost:8001/docs`
6. Now oas-tools needs special annotations to select the router and controller to use.
   * Just before `operationId: getItems` add the following property: `x-swagger-router-controller: itemRoute`
   * Just before `operationId: createItem` add the following property: `x-swagger-router-controller: itemRoute`
   * Just before `operationId: getItem` add the following property: `x-swagger-router-controller: itemRoute`
   * Just before `operationId: updateItem` add the following property: `x-swagger-router-controller: itemRoute`
   * Just before `operationId: deleteItem` add the following property: `x-swagger-router-controller: itemRoute`
8. There is another annotation that needs to be set:
   * OpenAPI Specification version 3 defines request's body in a different way, it is not a parameter as it is in Swagger version 2. Now requests bodies are defined in a section 'requestBody' which doesn't have name property, therefore it needs this property to work with your swagger-codegen generated controllers. Simply add to each requestBody secction the property 'x-name:' and the name of the resource.
   * Update the updateItem and createItem endpoints adding `x-name: item` just after the `requestBody` property, with the same indentation as description. 

9.  Run `npm run test`, currently only has one test, located at `/test/application.spec.ts` only checks the `/docs` endpoint. See how the test pass.

# Workshop step2
To start this step you need to accomplish the Workshop step1 section. The solution is already provided in branch `step1`.

1. `oas-tools` provides extra checks and configurations, stop the app and change the the `router: false` variable to `router:true` at `src/middlewares/swagger.ts`.
2.  Run the service again using `npm run dev`. You will see how the service fails with the following output:
   ```
    2019-04-03T10:15:16.720Z debug: Register: GET - /items
    2019-04-03T10:15:16.720Z debug:   GET - /items
    2019-04-03T10:15:16.721Z debug:     Spec-file does not have router property -> try generic controller name: itemsController
    2019-04-03T10:15:16.721Z debug:     Controller with generic controller name wasn't found either -> try Default one
    2019-04-03T10:15:16.721Z error:     There is no controller for GET - /items
   ```
3. `oas-tools` now, validates that the routes and controllers exists. Let's create the router first.
    * Create the file `itemRoute.ts` at `src/routes`
    * Add the Router and ayncHandler objects to the file:
  ```javasript
  import { Router } from "express";
  import { asyncHandler } from "../lib/asyncHandler";
  ```
    * We need to add the controllers. the following guide only covers the creation of the first controller, the following ones are going to be created by the students.

5. To complete the next step, the server already contains an ItemDao located at `src/dao` and models at `src/models` to simulate the storage of the items.
6. Create the getItemsHandler.ts at `src/controllers`.
   * Set the following code:
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

   * Update the itemRouter to include the controller:
      ```javascript
        export const getItems  = Router().use("/", asyncHandler(getItemsHandler, "getItems"));
      ```
    * Note: All the parameters and body objects are available in the `req.swagger.params.xxx.value` attribute.
7. start the server again and see how, errors doesn't appear and you are able to retrieve the list of items (now you will receive an empty array of Items).
8. Implement the rest of the endpoints. The solution is available at step2 branch.

# Workshop step3
We have the service running, but there is one thing left. TESTS!!!! in order to have a reliable service and have less error, we need to include unit tests.
1. We can do 2 kind of tests. Check the endpoints using `chai-http` and test the controllers itself(Not covered in this workshop). It would be nice if we prepare the test for the `ItemDAO`.
2. This workshop shows the creation of the test for one of the endpoints. Students will need to create the rest of the endpoints tests. The solution is available at `step3` branch.
3. Create the `item.route.spec.ts` in `test/routes`.
4. Add the following data inside:
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
5. Run `npm run test` and see how the tests for getItems endpoint works.
6. Add the tests for remmaining endpoints.
NOTE: Following TDD approach, we should create the tests first and development should be based on passing those tests.
7. Now, let's create the tests for the ItemDAO object
8. Create `item.dao.spec.ts` in `test/dao` folder.
9. Add the following content inside:
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
10. asdsad

