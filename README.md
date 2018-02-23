## api2go-resource

[![GoDoc](https://godoc.org/github.com/nathan-osman/api2go-resource?status.svg)](https://godoc.org/github.com/nathan-osman/api2go-resource)
[![MIT License](http://img.shields.io/badge/license-MIT-9370d8.svg?style=flat)](http://opensource.org/licenses/MIT)

This package serves as a bridge between [GORM](https://github.com/jinzhu/gorm) and [api2go](https://github.com/manyminds/api2go), reducing the amount of boilerplate code needed for implementing CRUD actions for GORM models.

### Features

Here are some of the features that api2go-resource provides:

- Works with your existing GORM models
- Enables filtering to be limited to specific fields
- Provides hooks to enable access control and normalization

### Usage

Let's suppose you have the following model definition:

```go
type Article struct {
    ID      int64
    Title   string
    Content string
}
```

In order to use the model with api2go, three important methods must be implemented:

```go
func (a *Article) GetName() string {
    return "articles"
}

func (a *Article) GetID() string {
    return strconv.FormatInt(a.ID, 10)
}

func (a *Article) SetID(id string) error {
    a.ID, _ = strconv.ParseInt(id, 10, 64)
    return nil
}
```

The next step is to create a `Resource` instance for the model:

```go
import "github.com/nathan-osman/api2go-resource"

// db is an instance of *gorm.DB

var articleResource = &resource.Resource{
    DB:   db,
    Type: &Article{},
}
```

This resource can now be registered with api2go:

```go
api := api2go.NewAPI("api")
api.AddResource(&Article{}, articleResource)
```

### Hooks

Hooks can be used for a number of different purposes.

For example, to make a resource read-only:

```go
func readOnly(action resource.Action, api2go.Request) error {
    switch action {
    case Create, Delete, Update:
        return errors.New("invalid action")
    }
    return nil
}

var articleResource = &resource.Resource{
    // ...
    GlobalHooks: []GlobalHook{readOnly},
}
```

To ensure that articles are always retrieved in alphabetical order:

```go
func alphabeticalOrder(c *gorm.DB, api2go.Request) *gorm.DB {
    return c.Order('title')
}

var articleResource = &resource.Resource{
    // ...
    GetHooks: []GetHook{alphabeticalOrder},
}
```

To remove any extra whitespace in an article title before saving:

```go
func trimTitle(obj interface{}, api2go.Request) {
    obj.Title = strings.TrimSpace(obj.Title)
}

var articleResource = &resource.Resource{
    // ...
    SetHooks: []SetHook{trimTitle},
}
```
