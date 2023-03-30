# Echolog
The [Echo](https://github.com/labstack/echo) middleware that contains functionality of requestid, recover and logs HTTP request/response details for traceability.

## Installation
Install echolog with:

```sh
go get -u github.com/Qiscus-Integration/echolog
```

## Usage
```go
package main

import (
	"net/http"

	"github.com/Qiscus-Integration/echolog"
	"github.com/labstack/echo/v4"
	"github.com/rs/zerolog/log"
)

func filter(c echo.Context) bool {
	return c.Request().RequestURI == "/healthcheck"
}

func main() {
	e := echo.New()
	// simple apply middleware without filter
	e.Use(echolog.Middleware())
	// with filter
	e.Use(echolog.Middleware("/healthcheck"))
	// add more filter
	e.Use(echolog.Middleware("/healthcheck", "foo", "bar"))
	
	// you can also do this
	e.Use(echolog.MiddlewareFilterFunc(filter))
	// or without filter
	// e.Use(echolog.MiddlewareFilterFunc(nil))
	// Output: {"level":"info","request_id":"9627a4a0-9d94-4ab6-844c-9599c0a15cd0","remote_ip":"[::1]:62542","host":"localhost:8080","method":"GET","path":"/","body":"","status_code":200,"latency":0,"tag":"request","time":"2023-02-19T14:07:37+07:00","message":"success"}

	e.GET("/", func(c echo.Context) error {
		ctx := c.Request().Context()
		log.Ctx(ctx).Info().Msg("hello world")
		// Output: {"level":"info","request_id":"9627a4a0-9d94-4ab6-844c-9599c0a15cd0","time":"2023-02-19T15:06:39+07:00","message":"hello world"}

		return c.String(http.StatusOK, "ok")
	})

	e.Logger.Fatal(e.Start(":8080"))

}

```