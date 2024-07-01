---
title: "Exploring PocketBase with Go: A Beginner's Guide"
seoTitle: "PocketBase with Go: Beginner's Guide"
seoDescription: "PocketBase: Compact, Go-based backend for small to mid-sized projects with real-time database and extensive auth options"
datePublished: Mon Apr 29 2024 11:11:24 GMT+0000 (Coordinated Universal Time)
cuid: clvkuzvd600030al49s6sarml
slug: exploring-pocketbase-with-go-a-beginners-guide
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1713870478386/9e978728-84f6-4b3b-9780-a5cfe2e0816a.png
tags: go, backend-as-a-service, pocketbase

---


I discovered [PocketBase](https://pocketbase.io) about a year ago and have used it as the backend for several small personal projects. It's a fantastic little backend-as-a-service that works well for small to mid-sized projects. The content for my [personal portfolio](https://richardwestmoreland.com) site is hosted on a PocketBase instance running on Fly.io.

### PocketBase Pros

#### Size

PocketBase is built on Go, so it all compiles to a single executable file. It's tiny! The PocketBase executable is around 35mb, in comparison to something like Strapi, which has a bundled size bigger than 2GB!

#### Authentication providers

Pocketbase offers pretty much every authentication provider that you can think of. It provides the possibility of manual authentication implementation (username, password) and 0Auth2.

![Some of the auth providers available](https://cdn.hashnode.com/res/hashnode/image/upload/v1714387932085/ae4fbe76-ba6c-4e76-9c3b-520be1f1d154.png align="center")

#### Real-time database functionality

PocketBase offers a real-time, firebase-like connection to the database. See updates on your front end in real time when you subscribe to real-time database events.

#### JavaScript and Dart SDKs are available

PocketBase has both Javascript and Dart SDKs for you to interface with in. your frontend. So whether you're building something for the web or something like Flutter (Dart), PocketBase has you covered and the docs are excellent.

#### It's free!

PocketBase is completely open-source and free. You just need a place to host it.

### PocketBase Cons

#### SQLite

While SQLite is an extremely fast and powerful little database solution, it doesn't scale horizontally. If you want to scale, buy more storage.

#### Not generally available

At the time of writing this, PocketBase is still considered "beta". And any version change is considered "breaking". I'll be excited when PocketBase reaches v1.0.0 / general availability.

### Extending PocketBase with Custom Routes

PocketBase allows you to add custom business logic and additional API routes. Documentation is available for expanding the framework using both Go and JavaScript. Here are a few examples of how to use PocketBase as a framework by enhancing its functionality with Go.

#### Validate data before creating a record

The `OnRecordBeforeCreateRequest` hook is triggered before each API record create request. If you wanted to do something like validate your submitted data before the record is created in the database, you'd incorporate this hook. Here's a simple example of validating a phone number as being from the USA:

```go
package main

import (
    "errors"
    "log"
    "regexp"

    "github.com/pocketbase/pocketbase"
    "github.com/pocketbase/pocketbase/core"
)

func main() {
    app := pocketbase.New()

    // Validate phone numbers for the "users" collection
    app.OnRecordBeforeCreateRequest("users").Add(func(e *core.RecordCreateEvent) error {
        phoneNumber, ok := e.Record.GetString("phone") // Assuming 'phone' is the field name
        if !ok {
            return errors.New("phone number field is missing")
        }
        
        // Regular expression to match US phone numbers (basic validation)
        if match, _ := regexp.MatchString(`^\(?([0-9]{3})\)?[-. ]?([0-9]{3})[-. ]?([0-9]{4})$`, phoneNumber); !match {
            return errors.New("invalid US phone number")
        }
        
        log.Println("Valid phone number:", phoneNumber)
        return nil
    })

    if err := app.Start(); err != nil {
        log.Fatal(err)
    }
}
```

Here you can see that we return an error if the phone number is not valid. Else we return nil, and the record will be created.

#### Register new API routes

You can also add your own API routes that perform custom logic. Here a new route is registered called `/hello/:name` :

```go
import (
    "log"
    "net/http"

    "github.com/labstack/echo/v5"
    "github.com/pocketbase/pocketbase"
    "github.com/pocketbase/pocketbase/core"
)

...

app.OnBeforeServe().Add(func(e *core.ServeEvent) error {
    e.Router.GET("/hello/:name", func(c echo.Context) error {
        name := c.PathParam("name")

        return c.JSON(http.StatusOK, map[string]string{"message": "Hello " + name})
    }, /* optional middlewares */)

    return nil
})
```

When we call this route with a name, for example, `/hello/richard` ; we'll get a 200 JSON response with `Hello richard` as our message.

### Give it shot

PocketBase is small but mighty. I highly recommend you head over to the [docs](https://pocketbase.io/docs/) and try it for yourself. If you have any questions about PocketBase, or if you'd like to see more articles about it, let me know in the comments!
