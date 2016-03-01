# Build process

The following command is used to produce docker image:
```
nildev build github.com/username/service
```

This command aggregates multiple different steps to do that:

1. Generate endpoint handler code
2. Generate endpoints routing code
3. Build binary
4. Build docker image

All this build process is wrapped up in `api-builder` container [here](https://github.com/nildev/api-builder). And `nildev build` command is just a cli interface to it.

Idea is that this `api-builder` container could be used to automate build process of final artifact - docker image. Then we could use it CI, CD process.

# Generate endpoint handler code

```
nildev io --sourceDir github.com/username/project --tpl simple-handlers --org nildev --ver v0.1.0
```

This command will walk through the root directory of `$GOPATH/github.com/username/project` and will generate endpoint handlers for each exported function. You can run this command on any project and as result you should find `gen_init.go` inside your project's root directory.

Generated code will look something like this:

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

In this generated file there are 3 important things:

1. Input and output types
2. HTTP Request handler
3. Route

Every time you modify your function `nildev` will regenerate this file.

# Generate endpoints routing code

In first step we did generate code which allows us to use our endpoint handlers in some API server. Previously generated `ge_init.go` file has public method `NildevRoutes() router.Routes` which you can use in your own server to integrate it.

Or you can use [`api-host`](https://github.com/nildev/api-host) server. This is minimal REST API server which provides:

* JWT support
* CORS support

No matter which option you will select the following command:
```
nildev r --services github.com/username/project --containerDir github.com/nildev/api-host --tpl simple-router --org nildev --ver v0.1.0
```

Will generate code that will integrate your API endpoint project with your API server. 

* `--containerDir` flag indicates which API server you want to use
* `--services x,y,z` is comma separated list of API services you want to build in
* `--tpl`, `--org` and `--ver` is template that should be used to generate code

You can run this command and as result in `--containerDir`, `gen` folder you will find `gen_init.go` file. This file will look something like this:

```
package gen

import (
	"github.com/nildev/lib/router"

	githubcomusernameproject "github.com/username/project"
)

func BuildRoutes() []router.Routes {
	routes := make([]router.Routes, 1)

	routes = append(routes, githubcomusernameproject.NildevRoutes())

	return routes
}
```

If we would have passed multiple services to `--services` flag here we would have multiple routes appended to `routes` slice.

Bottom line is that `BuildRoutes()` function is interface we can use now in our API server to register routes. As done [here](https://github.com/nildev/api-host/blob/master/endpoints/router.go#L38) in `api-host` project.

# Templates `--tpl`, `--ver`, `--org`

As you have noticed above commands takes these 3 flags. [This](https://github.com/nildev/templates) repository here hosts these templates. 

Bottom line is:

> Any file that has such comment at the top of the file can be used with nildev tool. 
```
//nildev:template organisation:template-name vX.Y.Z
```

So above commands with these flags:
```
... --tpl simple-handlers --org nildev --ver v0.1.0
... --tpl simple-router --org nildev --ver v0.1.0
```

Will use these two templates:

* [simple-handlers](https://github.com/nildev/templates/blob/master/rest/simple-handlers.tpl#L1)
* [simple-router](https://github.com/nildev/templates/blob/master/rest/simple-router.tpl#L1)

## How `nildev` locates these templates

Current implementation just recursively iterates over `$GOPATH/src` directory and looks for files that has that comment at their first line. Meaning that if you want to use different template just create it anywhere in `$GOPATH` and add required comment.

# Integrate with `go: generate`

In source code where your endpoint function lives add these lines:
```
//go:generate nildev io --sourceDir github.com/username/project --tpl simple-handlers --org nildev --ver v0.1.0
//go:generate nildev r --services github.com/username/project --containerDir github.com/nildev/api-host --tpl simple-router --org nildev --ver v0.1.0
```

And then you can run:
```
go generate
```

In this way you can abstract generation process. 

# Building binary

This is self explanatory. Once we have all code generated we just produce binary. This is done  [here](https://github.com/nildev/api-builder/blob/master/build.sh#L34) in `api-builder`.

# Building docker image

If your API server has `Dockerile` in it's root directory, like [this](https://github.com/nildev/api-host/blob/master/Dockerfile) one here at `api-host`, then `api-builder` will build and image. This bit is done [here](https://github.com/nildev/api-builder/blob/master/build.sh#L44)

# Summary

As you see this whole process is fully customizable. But idea is that `nildev` should have default one which would cover 80% of use cases. Ideally developer would have to run only this:

```
nildev build github.com/my/project
```

And as result docker image with fully working REST API endpoint would be produced.


