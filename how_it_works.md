# How it works

General flow is as follows.

1. You create repository which contains your endpoint function
2. `nildev` tool parses endpoint function and generates required integration code
3. `nildev` uses built `api-builder` container to integrate selected API server with your endpoint function. It produces binary.
4. `nildev` builds a docker image which has your service binary on it

# Parsing endpoint functions

When you run `nildev build github.com/your_org/your_service`, `nildev`goes through all `go` code in root directory of your project and parses all exported functions. After that it generates code which contains `http.HandlerFunc` functions for each of your function.

Create empty project in `$GOPATH`, for example `$GOPATH/src/github.com/your_org/servicex` and inside create file `my-service.go`. Add there any number of exported functions with any kind of input parameters. Then run `nildev` to generate missing code:

```
nildev io --sourceDir=$GOPATH/src/github.com/your_org/servicex
```

Inside `$GOPATH/src/github.com/your_org/servicex` you will find generated file `gen_init.go`. It will look something like this:

```
package authx

// THIS IS AUTO GENERATED FILE - DO NOT EDIT!
import (
		 "errors"
		 "github.com/dgrijalva/jwt-go"
		 "github.com/gorilla/context"
		 "github.com/gorilla/mux"
		 "github.com/nildev/lib/registry"
		 "github.com/nildev/lib/router"
		 "github.com/nildev/lib/utils"
		 "log"
		 "net/http"
		 "strconv"
)

type (
	// InputRegister struct
	InputRegister struct {
		Provider string `json:"provider,omitempty"`
		UserName string `json:"userName,omitempty"`
	}

	// OutputRegister struct
	OutputRegister struct {
		Result bool `json:"result,omitempty"`
		Err error `json:"err,omitempty"`
	}
)

// RegisterHandler HTTP request handler
func RegisterHandler(rw http.ResponseWriter, r *http.Request) {
	returnCode := http.StatusOK
    var requestData map[string]string
    requestData = mux.Vars(r)
    log.Infof("Request vars [%+v]", requestData)
    

	reqDTO := &InputRegister{}
	utils.UnmarshalRequest(r, reqDTO)

    cvprovider, convErr := GetVarValue(requestData, "provider", "string")
	if convErr != nil {
		returnCode = http.StatusInternalServerError
		utils.Respond(rw, convErr, returnCode)
		return
	}

	if cvprovider != nil  {
		reqDTO.Provider = cvprovider.(string)
	}
	
    cvuserName, convErr := GetVarValue(requestData, "userName", "string")
	if convErr != nil {
		returnCode = http.StatusInternalServerError
		utils.Respond(rw, convErr, returnCode)
		return
	}

	if cvuserName != nil  {
		reqDTO.UserName = cvuserName.(string)
	}

	result, err := Register(reqDTO.Provider,reqDTO.UserName)

	if err != nil {
		returnCode = http.StatusInternalServerError
	}

	respDTO := &OutputRegister{
		Result:result,
		Err:err,
	}

	utils.Respond(rw, respDTO, returnCode)
}

// NildevRoutes returns routes to be registered
func NildevRoutes() router.Routes {
	routes := router.Routes{
		BasePattern: "/api/v1",
		Routes: make([]router.Route, 1),
	}

	routes.Routes[0] = router.Route{
		Name:        "github.com/nildev/authx:Register",
		Method:      "POST",
		Pattern:     "/custom-register/{provider}",
		Protected:   false,
		HandlerFunc: RegisterHandler,
		Queries:     []string{
		    "userName",
		    "{userName:[a-z]+}",
		},
	}
	return routes
}

func GetVarValue(data map[string]string, name, typ string) (interface{}, error) {
	// if value exists in variables
	if val, ok := data[name]; ok {
		switch typ {
		case "string":
			return val, nil
		case "*string":
			return &val, nil
		case "*int":
			i, err := strconv.Atoi(val)
			return &i, err
		case "int":
			i, err := strconv.Atoi(val)
			return i, err
		case "int8":
			i, err := strconv.ParseInt(val, 10, 8)
			return int8(i), err
		case "*int8":
			i, err := strconv.ParseInt(val, 10, 8)
			icst := int8(i)
			return &icst, err
		case "int16":
			i, err := strconv.ParseInt(val, 10, 16)
			return int16(i), err
		case "*int16":
			i, err := strconv.ParseInt(val, 10, 16)
			icst := int16(i)
			return &icst, err
		case "int32":
			i, err := strconv.ParseInt(val, 10, 32)
			return int32(i), err
		case "*int32":
			i, err := strconv.ParseInt(val, 10, 32)
			icst := int32(i)
			return &icst, err
		case "int64":
			i, err := strconv.ParseInt(val, 10, 64)
			return i, err
		case "*int64":
			i, err := strconv.ParseInt(val, 10, 64)
			return &i, err
		case "float32":
			i, err := strconv.ParseFloat(val, 32)
			return float32(i), err
		case "*float32":
			i, err := strconv.ParseFloat(val, 32)
			icst := float32(i)
			return &icst, err
		case "float64":
			i, err := strconv.ParseFloat(val, 64)
			return i, err
		case "*float64":
			i, err := strconv.ParseFloat(val, 64)
			icst := float64(i)
			return &icst, err
		case "bool":
			b, err := strconv.ParseBool(val)
			return b, err
		case "*bool":
			b, err := strconv.ParseBool(val)
			return &b, err
		default:
			return nil, errors.New("Value is complext type!Expected only primitives!")
		}
	}

	return nil, nil
}

```

As you see it does not do much, just simple wiring. But it saves times as I do not need to write this code. We use here `gorilla/*` packages but if you need to have this wiring done differently you can change templates.

# Defining endpoints

As we already established defining endpoint means creating an exported function. But apart from that there are couple comments that helps you to define:

* HTTP method
* Query parameters
* Path

## `@method` tag

Any valid HTTP method will fit here. This will end up as `Method` in `gorilla/mux`.

## `@path` tag

Value of this tag will end up as `Path` in `gorilla/mux`. Documentation is [here](http://www.gorillatoolkit.org/pkg/mux#Route.Path).

## `@query` tag

Value of this tag will end up as `Queries` in `gorilla/mux`. Docs [here](http://www.gorillatoolkit.org/pkg/mux#Router.Queries).

## Optional parameter

Some parameters can be optional. To define such parameter make it pointer. For example:

```
function Endpoint(optional *string) (err error) {...}
```

## Protected endpoints

To protect your resource and allow it for authenticated and authorized users add `@protected` tag. For example:

```
// @protected
function Endpoint(optional *string) (err error) {...}
```
When this function will be parsed, JWT middleware will be added for this route. All requests will have to have JWT token included in `Authentication` header. This is built in default [`api-host`](https://github.com/nildev/api-host) server.

Besides of that if you need JWT `Claims` in your endpoint function just add `user` of type `registry.User` parameter to your endpoint signature:

```
import "github.com/nildev/lib/registry"

function Endpoint(user registry.user) (err error) {...}
```

If you will generate code again:
```
nildev io --sourceDir=$GOPATH/src/github.com/your_org/servicex
```

You will see that data from JWT `Claims` will be passed for your endpoint function.

# `api-host` servers

# `api-builder` container