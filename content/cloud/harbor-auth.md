---
title: "harbor 源码分析： 用户认证"
date: 2017-11-22T11:14:50+08:00
tags: ["docker", "harbor", "authenticate"]
categories: 云计算
slug: harbor-authenticate.md
type: post
---


## 接口定义
代码在 `src/ui/auth/authenticator.go`  

```go
// Authenticator provides interface to authenticate user credentials.
type Authenticator interface {

	// Authenticate ...
	Authenticate(m models.AuthModel) (*models.User, error)
}
```

接口中定义了函数 Authenticate，只要对应的结构体实现了该函数，就能完成认证。  
我们通过其中的数据库认证方式，来简单分析。

## 接口实现
代码在 `src/ui/auth/db/db.go`

```go
// Auth implements Authenticator interface to authenticate user against DB.
type Auth struct{}

// Authenticate calls dao to authenticate user.
func (d *Auth) Authenticate(m models.AuthModel) (*models.User, error) {
	u, err := dao.LoginByDb(m)
	if err != nil {
		return nil, err
	}
	return u, nil
}
```

## 接口的注册
代码在 `src/ui/auth/db/db.go`

```go
func init() {
	auth.Register("db_auth", &Auth{})
}

```

初始化的时候，将自身注册到 auth 中，这里我们可以继续跟进 auth.Register 这个函数

```go
var registry = make(map[string]Authenticator)

// Register add different authenticators to registry map.
func Register(name string, authenticator Authenticator) {
	if _, dup := registry[name]; dup {
		log.Infof("authenticator: %s has been registered", name)
		return
	}
	registry[name] = authenticator
}
```

这里很清楚的可以看到， registry 是一个 map，最终会大概成为这样：

```json
{
    "db_auth": &Auth{},
    "ldap_auth": &Auth{},
    "uaa_auth": &Auth{}
}
```

## 接口调用
`db_auth`, `ldap_auth`, `uaa_auth` 中都实现了 Authenticate 这个函数。
那么使用的时候，我们就可以通过配置文件、环境变量之类的值，来确定我们使用的认证方式，
然后调用对应的认证方法。

代码在 `src/ui/auth/authenticator.go`

```go
// Login authenticates user credentials based on setting.
func Login(m models.AuthModel) (*models.User, error) {

	authMode, err := config.AuthMode()
	if err != nil {
		return nil, err
	}
	if authMode == "" || m.Principal == "admin" {
		authMode = "db_auth"
	}
	log.Debug("Current AUTH_MODE is ", authMode)

	authenticator, ok := registry[authMode]
	if !ok {
		return nil, fmt.Errorf("Unrecognized auth_mode: %s", authMode)
	}
	if lock.IsLocked(m.Principal) {
		log.Debugf("%s is locked due to login failure, login failed", m.Principal)
		return nil, nil
	}
	user, err := authenticator.Authenticate(m)
	if user == nil && err == nil {
		log.Debugf("Login failed, locking %s, and sleep for %v", m.Principal, frozenTime)
		lock.Lock(m.Principal)
		time.Sleep(frozenTime)
	}
	return user, err
}
```