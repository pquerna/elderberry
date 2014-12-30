# elderberry

`elderberry` is an experiment in Go web frameworks.  It doens't work yet.

# README driven development.

# Example

## Using EB Server

You previously ran `elderberry github.com/pquerna/example/mything github.com/pquerna/example/generated_server`, and it generated the `generated_server` package. This `main` lets you use reflection for development, and in production builds asserts that the reflect'ed handlers are equal.

```go
package main

import (
	eb "github.com/pquerna/elderberry"
	"github.com/pquerna/example/generated_server"
	"github.com/pquerna/example/mything"
)

// could be an enviroment variable or flag
const devenv = false

func main() {
	var server eb.Server
	if devenv {
		server = eb.NewDevelopmentServer()
	} else {
		server = generated_server.NewServer()
	}

	// Register adds handlers to the Development server (which uses reflection
	server.Register(mything.BuildHandler())

	// If the generated code does not match the registered handlers here when using `generated_server`, elderberry will panic().
	server.ListenAndServe()
}

```

# example handlers and middlewares.

```go

package mything

import (
	eb "github.com/pquerna/elderberry"
	"net/http"
)

func BuildHandler() []eb.Handler
	return []eb.Handler{eb.Handler{
		Route: eb.Method("POST").Path("/v1/<int:tenantId>/email"),
		Middleware: []eb.Middleware{
			authenticateTenant,
			// this is a runtime initilized middlware, EB will call authorizeTenant with any parameters
			// and then add the return value to the middleware chain.
			eb.Middleware(authorizeTenant, "tenant.email.send"),
			rateLimit,
		},
		Handler: sendEmail,
	}}
}

type Tenant struct {
	Id int	
}

// Based on the TenantID in the URL, return a Tenant object or an error.
// would also likely validate your tenantId, bearer token, authenication, etc here.
func authenticateTenant(req *http.Request, tenantId int) (*Tenant, error) {
	return &Tenant{Id: tenantId}, nil
}

type authzFunc func (t *Tenant) error
// Is this tenant allowed to do this operation?
func authorizeTenant(capabilityName string) authzFunc {
	return func(t *Tenant) error {
		// use capabilityName to check if this combination of user/tenant can do this operation.
		if t.Id == 1 {
			// returning a bare error creates a 500
			// If the error implements ErrorCode() int via an
			// interface upgrade, it will render that as the HTTP response code.
			return errors.New("this tenantId is disabled.")
		}
		return nil
	}	
}

func rateLimit(req *http.Request, t *Tenant) error {
	 // implement rate limiting here
	 return nil
}

func sendEmail(rw http.ResponseWriter, req *http.Request) {
	// send your email
	rw.Write([]byte("success."))
}

```

