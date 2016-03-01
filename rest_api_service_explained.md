# REST API service explained

Before you begin, please follow [Setting up environment](setting_up_environment.md) section if you haven't done that yet.

In this guide we will create REST API service and will produce docker image. This guide is a detailed version of [this guide](rest_api_in_less_than_3_minutes.md).

## Setup project skeleton

To setup project skeleton we will use `nildev` tool. Under the hood `nildev` uses [project](https://github.com/nildev/project). 
We will start by creating `authx-service.json` file at home directory:

```
wget https://raw.githubusercontent.com/nildev/prj-tpl-basic-api/master/project.config.json -O ~/authx-service.json
```

Content of `authx-service.json` should look like this:
```
{
  "ApiName":"authx",
  "ApiNameAllCapital":"AUTHX",
  "ProjectPath":"github.com/your_org/authx",
  "ApiNameExported":"Authx",
  "TableName":"sessions",
  "Org":"your_org",
  "FullAuthor":"Your Name <your@email.com>",
}
```

Now create new project:

```
nildev create --config=~/authx-service.json github.com/your_org/authx 
```

Command above will clone [this](https://github.com/nildev/prj-tpl-basic-api) project template and will replace all variables with values you have in `authx-service.json`. Last argument `github.com/your_org/authx` indicates where project will be created. This path is relative to `$GOPATH` directory.

If you need you can create your own project template and pass it to `create` command:
```
nildev create --config=~/authx-service.json \
              --template="git@github.com:your_org/your-prj-tpl.git" \
              github.com/your_org/authx 
```

Do not forget to include variables in `your-service-config.js` once you will add it to your project template.

## Project template `prj-tpl-basi-api`

This template has everything you need to start building REST API. Moreover it includes code required to persist/read data from MongoDB, sample endpoints, docker-compose configurations.

## API Endpoint functions

As I already mentioned, project template we have used has everything prepared and ready to be build. But let's open `authx.go` (if you have named your service differently name of file will be `your_name.go`) and go through it.

In file you will find two functions:
```
func Register(provider string, userName string, email *string) (result bool, msg string, err error)
func ProtectedResource(user registry.User) (result bool, err error)
```

These two functions is our endpoint handlers. Once we will build our final service binary it will serve two endpoints which will be executing these functions. Bottom line is that every exported function in root package will become and endpoint. So essentially to expose REST API endpoint all you have to do is to export function:

```
function DoSomething() (err error){
    return nil
}
```

That's it, once you build a binary you will have `GET /do-something` endpoint available.

## Input and Output

> function signature parameters is our endpoint *input* and all return parameters is *output*

For example `Register` method here has 2 input and 2 output parameters. We define that `provider` will be passed in request path, that endpoint will accept `POST` and that query string must contain `userName`. We defined that our response will be JSON object will contain `result` and `err`. If value is empty key will be omited and not included. 
```
// Register method
// @path /custom-register/{provider}
// @method POST
// @query {userName:[a-z]+}
func Register(provider string, userName string) (result bool, err error) {

	// your logic here

	return true, nil
}
```

You can have any type of input or output parameters ([here](https://github.com/nildev/lib/blob/master/codegen/parser_test.go) you can see test cases). Output parameters are being wrapped in DTO and returned to client. This means that your function signutes becomes documentation for your API user.

As you have noticed in comments we have couple of tags which are used to configure our endpoint.

### `@path` 

Within this tag you can configure your endpoint path. Under the hood it uses `gorilla/mux` (docs here). Every endpoint will be prepended with base path `/api/v[0-9]{1}`. Meaning that:
```
// @path /custom-register/{provider}
``` 

Will be available at:
```
/api/v1/custom-register/{provider}
```

In path you can use `{variable:[a-z]}` as explained in `gorilla/mux` documentation. These variable names should matched signature parameter names:
```
// @path /custom/{provider}
function Endpoint(provider string) (err error){
}
```

### `@method`

By default if this tag is not presented endpoint will be available under `GET` method. Use this tag if you want to allow different method or multiple methods:
```
// @method POST DELETE
function Endpoint(provider string) (err error){
}
`
```

### `@query`

With query you can define what parameters can be passed in query string, which are required to match route and which are optional. Under the hood we use again `gorilla/mux`.
For example:
```
// @query {userName:[a-z]+} {email:[a-z]+}
function Endpoint(userName string, email string, required int, optional *bool) (err error){
}
`
```

If you do not need any regular expression checks on your parameters then you can leave out `@query` tag. It will be taken from function signature. Pointer parameters will be made optional and if won't be passed value will be `nil`.

You can have `@query` tag for `POST` methods as well. Just if you will add in function signature parameter which will not be included in `path` or `query` then this parameter will be expected in request body.

### `@protected`

Add this tag if you want to protect your endpoint. Once added this will enabled `JWT` middleware and valid token will be expected in `Authorization` header. How token is generated is another topic, but [this]() service here does that. This `Auth` service is built with `nildev`.

For protected endpoints you can add a reserved parameter `user` in your function signature:
```
import "github.com/nildev/lib/registry"

// @protected
function Endpoint(user registry.User) (err error){
}
```

If valid `JWT` token will be passed `user` will allow to access `Claims`. 

## Actual function code

Once your endpoint function is defined you can focus on code that matters. Inside of this function we should write code that is unique to your endpoint. Connect to DB, access other services, transform data and so fourth. There are no limitations. 

Let's change our service to accept third optional `email` parameter. We do that by making `email` of `*string` type. We also have added one more response parameter `msg`. After changes our service looks like this:
```
// Register method
// @path /custom-register/{provider}
// @method POST
// @query {userName:[a-z]+}
func Register(provider string, userName string, email *string) (result bool, msg string, err error) {
	if email != nil {
		msg = *email + ":" + userName
	} else {
		msg = userName
	}
	return true, msg, nil
}
```

This endpoint now will return as response either `result` and `msg`:
```
{"result":true, "msg":"em:ss"}
```
Or just `err`:
```
{"err":"error msg"}
```

Also `msg` value will depend either we will send in body `email` or not:
```
{"email":"email@sss.lt"}
```

## Building container

Now we can build our service. Building contains couple steps:

1. Parse exported functions in root directory of your service
2. Generate required integration code (Response, request objects, routes etc.)
3. Fetch api-server that you want to use as host for your service
4. Build binary
5. Build docker image

All these parts we will cover in next chapters. For now just run the following:

```
nildev build github.com/nildev/authx
```

Result should be a docker image. Check if it is there:
```
docker images
```

## Running your service

Once our image is ready we can run our service. Let's create `local.env` file with content:
```
ND_DATABASE_NAME=Sessions
ND_MONGODB_URL=mongodb://YOUR_DOCKER_MACHINE_IP:27017/nildev
ND_ENV=dev
```

`YOUR_DOCKER_MACHINE_IP` is IP of machine where docker is running. If you are using `docker-machine` you can check it by running `docker-machine ip your_machine_name`.

Each environment will have it's own `*.env` file. We will use it to set some secrets (tokens etc.) in other environments.

Once file is created, we can run our service docker containers:
```
docker-compose -f docker-compose.yml up -d
```

If no errors are present it means your service is up and running, you can try to access it:
```
# no email passed
curl -X POST http://YOUR_DOCKER_MACHINE_IP/api/v1/custom-register/github?userName=nildev -v
# with email
curl -X POST -d'{"email":"your"}' http://YOUR_DOCKER_MACHINE_IP/api/v1/custom-register/github?userName=nildev -v
```

You should be getting different results for each request.


