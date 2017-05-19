# koa-swagger-ui

Still missing:
- [x] convert route outputs into swagger
- [ ] finish converting from joi-to-json-schema to [joi-to-swagger](https://github.com/ChiperSoft/joi-to-swagger) (there's no npm link, but there is a github link)
- [ ] make tests to demonstrate
- [ ] find out why swagger-cli throws validate errors on anything I create.



A node module for generating [Swagger 2.0](http://swagger.io/) JSON
definitions from existing [koa-joi-router](https://github.com/koajs/joi-router)
routes.

## Example

```js

const SwaggerAPI = require('joi-router-swagger-docs').SwaggerAPI;
const Router = require('koa-joi-router');
const Joi = Router.Joi;
const router = router();

router.get('/demo',{
   validate: {
      query: {
        name: joi.string().max(100).description('new user name')
      }
    },
    handler: async (ctx) => {
      ctx.status = 200;
      ctx.body = {status: 1, msg: 'hello world'};
    },
    swagger: {
      tags: ['demo'],
      responses: {
        200: {
          description: '查询域名',
          schema: joi.object({
            status: joi.number().integer().default(1),
            server: joi.string().default('js.cool')
          }).label('serverSuccess')
        }
      }
    }
  });

router.get('/signup', {
  validate: {
    type: 'json',
    body: {
      name: Joi.string().max(100).description('new user name')
    },
    output: {
      200: {
        body: {
          userId: Joi.string().description('newly created user id')
        }
      }
    }
  },
  handler: function*() {
    // ...
  }
});

router.get('/about/:versionId', {
  swagger: {
    description: 'Anything in swagger is passed directly onto the swagger object for that path',
    parameters: [
      {name: 'versionId', in: 'path', description: 'this is a good way to get other items onto swagger object.'}
    ]
  },
  handler: function*() {
    // ...
  }
})

//if you use swagger-ui you will want path parameters so people can use the 'try it out' functionality, despite the fact that koa-joi-router doesn't support them
router.get('/user/:id/friends/:friendId', {
  validate: {
    path: Joi.object().keys({
      id: Joi.string().alphanum().max(24).description('id of user').required(),
      friendId: Joi.string().alphanum().max(24).description('id of user\'s friend'),
    })
    output: {
      200: {
        body: {
          userId: Joi.string().description('The friend of the user. they are pretty cool.')
        }
      }
    }
  },
  handler: function*() {
    // ...
  }
});

swaggerAPI = new SwaggerAPI();
swaggerAPI.addJoiRouter(router);

let spec = swaggerAPI.generateSpec({
  info: {
    title: 'Example API',
    description: 'API for creating and editing examples',
    version: '1.1'
  },
  basePath: '/api/v1'
});

console.log(JSON.stringify(spec, null, '  '));
```

## API

### new SwaggerAPI()

Creates a new SwaggerAPI instance.

### swaggerAPI.addJoiRouter(router, options)

Add a joi-router instance to the API. The router should already have all its
routes set up before calling this method (which pulls the route definitions
from the router's `.routes` property).

Options:
- prefix: Prefix to add to Swagger path

### swaggerAPI.generateSpec(baseSpec)

Create a Swagger specification for this API. A base specification should be
provided with an `info` object (containing at least the `title` and `version`
strings) and any other global descriptions.

## Thanks for starting this:

[Pebble Technology!](https://www.pebble.com)

## License

[MIT](https://github.com/paul42/joi-router-swagger-docs/blob/master/LICENSE)
