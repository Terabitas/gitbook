# About

**nildev** is a set of tools and libraries that aims to boost up your productivity while building `Golang` services. We do that by allowing you to focus on code that matters and absorbing everything else. In other words **nildev** is **all those other things you do not want to waste your time on**.

# How it's done ?

Long story short - `nildev` generates boilerplate code and allows you to focus on endpoint logic. For example the following piece of code:

```
package servicex // import "github.com/username/servicex"

import (
	"github.com/nildev/lib/registry"
)

// The most basic endpoint
func MyEndpoint(num int, param string) (result bool, paramOne string, err error) {
    return true, "calculated-string", nil
}

// Register method
// @path /custom-register/{provider}
// @method POST
// @query {userName:[a-z]+}
func Register(provider string, userName string) (result bool, err error) {
    result = true
    err = nil

    return true, nil
}

// ProtectedResource method which should return only if user passes valid JWT token
// nildev will set `user` variable and will set to it whatever is in JWT Claims token
// @path /protected
// @protected
func ProtectedResource(user registry.User) (result bool, err error) {
    return true, nil
}
```

After running:

```
nildev build github.com/username/servicex
```

Will result in minimal docker container with binary on it which will contain fully working REST API with the following features:

* HTTP server with endpoints `POST /custom-register/{provider}`, `GET /protected` and `GET /my-endpoint`
* JWT middleware enabled for `/protected` endpoint
* CORS middleware enabled

Whole idea is to allow developer to focus on domain logic and delegate the rest to `nildev`. This code generation and building process is fully customizable and every single bit of it can be modified. But as `nildev` is open source project, aggregated community experience would result to recommended configuration which in 80% of use cases should be enough.


# Where to start ?

Start with [Setting up environment](setting_up_environment.md), then continue with [REST API in less than 3 minutes](rest_api_in_less_than_3_minutes.md) and [How it works](how_it_works.md) chapters. This should be enough to get you up running so you could start building your own API services.

