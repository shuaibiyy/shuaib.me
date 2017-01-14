+++
date = "2017-01-14T02:42:18+01:00"
description = ""
title = "Threading a DB connection through a Go API Server"

+++

In this post, I'll walk through code snippets for a Golang service, showing how a database connection can be passed around from its initialisation all the way to HTTP handlers. The service exposes 3 endpoints. It uses the [pgx](https://github.com/jackc/pgx) library for persistence, [gorilla/mux](https://github.com/jackc/pgx) for routing, [urfave/negroni](https://github.com/urfave/negroni) for HTTP middleware, and [Auth0](https://auth0.com/) for authentication.

This first snippet shows some very standard code for database initialisation.
```
import (
	"fmt"
	_ "github.com/jackc/pgx/stdlib"
	"github.com/jmoiron/sqlx"
	"log"
	"os"
)

func Init() *sqlx.DB {
	db, err := sqlx.Open("pgx",
		fmt.Sprintf("postgres://%s@%s:5432/postgres?sslmode=disable",
			os.Getenv("DB_USER"),
			os.Getenv("DB_SERVER")))
	if err != nil {
		log.Fatal(err)
	}
	if err := db.Ping(); err != nil {
		log.Fatal(err)
	}
	return db
}
```
The next snippet shows how the service's routes are defined.
```
type Route struct {
	Name    string
	Method  string
	Pattern string
	Handler HandlerWithDB // [1]
}

type Routes []Route
var routes = Routes{
	Route{
		"IndexTags",
		"GET",
		"/tags",
		handlers.IndexTags,
	},
	Route{
		"ShowTag",
		"GET",
		"/tags/{slug}",
		handlers.ShowTag,
	},
	Route{
		"SecuredCreateTag", // [2]
		"POST",
		"/tags",
		handlers.CreateTag,
	},
}
```
[1] This is the type of the API's handlers, and is defined as follows:

	type HandlerWithDB func(*sqlx.DB) http.HandlerFunc

Which means our API handlers end up looking like this:
```
var ShowTag = func(dbConn *sqlx.DB) http.HandlerFunc {
	return func(w http.ResponseWriter, r *http.Request) {
		slug := mux.Vars(r)["slug"]
		tag, err := db.SelectTag(dbConn, slug)
		if err != nil {
			http.Error(w, err.Error(), http.StatusNotFound)
			return
		}
		json.NewEncoder(w).Encode(tag)
	}
}
```

[2] The __SecuredCreateTag__ endpoint will require user authentication. The presence of the __Secure__ prefix in the handler's name will let the router creation function know that the handler should be wrapped by a higher-order function that enforces authentication.

The next snippet shows how the routers defined above are used to create a [mux](www.gorillatoolkit.org/pkg/mux) router:
```
import (
	"github.com/gorilla/mux"
	"github.com/jmoiron/sqlx"
	"github.com/urfave/negroni"
	"strings"
	"net/http"
)

func NewRouter(db *sqlx.DB) *mux.Router {
	r := mux.NewRouter().StrictSlash(true)
	for _, route := range routes {
		n := strings.ToLower(route.Name)
		if strings.HasPrefix(n, "secure") { // [3]
			r.Handle(route.Pattern, auth.SecuredRoute(db, route.Handler)).Methods(route.Method).Name(route.Name)
		} else {
			r.Handle(route.Pattern, route.Handler(db)).Methods(route.Method).Name(route.Name)
		}
	}
	return r
}


// [4]
var SecuredRoute = func(db *sqlx.DB, handler HandlerWithDB) http.Handler {
	return negroni.New(negroni.HandlerFunc(jwtMiddleware.HandlerWithNext), NegroniWrapper(db, handler)) // [5]
}
```
[3] Of note is the check for the word __secure__ in the handler's name. The use of a string to determine if an endpoint should be secured is at best an arguable thing to do. However, for my  inconsequential service, I can still sleep peacefully at night knowing fully well what I've done.

[4] The __SecuredRoute__ function uses [Negroni](https://github.com/urfave/negroni) to wrap __jwtmiddleware__ from [Go JWT middleware](https://github.com/auth0/go-jwt-middleware) by Auth0. Auth0 is an awesome service for offloading all of your authentication concerns <3

[5] The __New__ function from Negroni creates a middleware stack that can consist only of Negroni handlers, and we are passing it a __HandlerWithDB__ function, which is our type that does not implement the Negroni __Handler__ interface. Negroni provides a __Wrap__ function, but it is only able to wrap __http.Handler__ functions. In order to use Negroni, we have to write a wrapper for __HandlerWithDB__:
```
import (
	"github.com/jmoiron/sqlx"
	"github.com/urfave/negroni"
	"net/http"
)

type HandlerWithDB func(*sqlx.DB) http.HandlerFunc

// [6]
func (f HandlerWithDB) ServeHTTP(db *sqlx.DB, w http.ResponseWriter, r *http.Request) {
	g := f(db)
	g(w, r)
}

func NegroniWrapper(db *sqlx.DB, handler HandlerWithDB) negroni.Handler {
	return negroni.HandlerFunc(func(rw http.ResponseWriter, r *http.Request, next http.HandlerFunc) {
		handler.ServeHTTP(db, rw, r)
		next(rw, r)
	})
}
```
[6] We also have to define a __ServeHTTP__ function that takes a database connection as an additional argument.

Finally, our main function to tie it all together:
```
import (
	"net/http"
	"os"
)

func main() {
	dbConn := db.Init()
	router := NewRouter(dbConn)
	log.Fatal(http.ListenAndServe(":8080", router))
}
```
This is my first Go service. I hope you'll be generous enough to give feedback if you have any.
