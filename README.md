# elderberry

`elderberry` is an experiment in Go web frameworks.  It doens't work yet.

# README driven development.

# Example

## Using Runtime Reflection Server

```go
package main
import (
	eb "github.com/pquerna/elderberry"
	"github.com/pquerna/example/mything"
)

func main() {
	server := eb.ReflectServer()
	mything.Register(server)
	// TOOD: ports and shit.
	server.ListenAndServe()
}

```

## Using Generated Server

You previously ran `elderberry github.com/pquerna/example/mything github.com/pquerna/example/ebserver`, and it generated the `ebserver` package.

```go
package main
import (
	"github.com/pquerna/example/ebserver"
)

func main() {
	server := ebserver.Server()
	// your handlers are already registered.
	// TOOD: ports and shit.
	server.ListenAndServe()
}

```

```go

package mything

import (
	eb "github.com/pquerna/elderberry"
	"net/http"
)

func Register(server *eb.Server)
	server.Add(eb.Handler{
		Route: eb.Method("POST").Path("/v1/<int:tenantId>/email"),
		Middleware: []eb.Middleware{
			authenticateTenant,
			authorizeTenant("tenant.email.send"),
			rateLimit,
		},
		Handler: sendEmail,
	})
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

