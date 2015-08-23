---
layout: post
title: "go installation in mac using brew "
published: true
comments: true
tags: go mac brew
---

I wnated to get my hands dirty with go and so i thought i would install it in mac.So as usual i was trying to install  using brew.The installation went smooth but whenever i was trying to install a package i was thrown an error as below 

```
Pradheep ~ $ go get gopkg.in/mgo.v2
package gopkg.in/mgo.v2
	imports bytes: unrecognized import path "bytes"
package gopkg.in/mgo.v2
	imports crypto/hmac: unrecognized import path "crypto/hmac"
package gopkg.in/mgo.v2
	imports crypto/md5: unrecognized import path "crypto/md5"
package gopkg.in/mgo.v2
	imports crypto/rand: unrecognized import path "crypto/rand"
package gopkg.in/mgo.v2
	imports crypto/sha1: unrecognized import path "crypto/sha1"
package gopkg.in/mgo.v2
	imports encoding/base64: unrecognized import path "encoding/base64"
package gopkg.in/mgo.v2
	imports encoding/binary: unrecognized import path "encoding/binary"
package gopkg.in/mgo.v2
	imports encoding/hex: unrecognized import path "encoding/hex"
package gopkg.in/mgo.v2
	imports encoding/json: unrecognized import path "encoding/json"
package gopkg.in/mgo.v2
	imports errors: unrecognized import path "errors"
package gopkg.in/mgo.v2
	imports fmt: unrecognized import path "fmt"
package gopkg.in/mgo.v2
	imports io: unrecognized import path "io"
package gopkg.in/mgo.v2
	imports math: unrecognized import path "math"
package gopkg.in/mgo.v2
	imports net/url: unrecognized import path "net/url"
package gopkg.in
.....
```

The problem is that both the GOPATH and GOROOT was not set properly and PATH was not updated correctly.After several trial and error methods and of course at last google to the rescue i was able to finally get it working

For bash put the below in ~/.bashrc

```
# don't forget to change your path correctly!

export GOPATH=$HOME/golang
export GOROOT=/usr/local/opt/go/libexec
export PATH=$PATH:$GOPATH/bin
export PATH=$PATH:$GOROOT/bin

```

For zsh which is the default shell in mac (~/.zshrc)

```

# don't forget to change your path correctly!

export GOPATH=$HOME/golang
export GOROOT=/usr/local/opt/go/libexec
export PATH=$PATH:$GOPATH/bin
export PATH=$PATH:$GOROOT/bin

```

You can of course change the GOPATH to any of your project specific workspace.Just update the corresponding path in your folder you are interested.

Just make sure that there is no space in the folder path as i find it causes too much problem :(


I found the above solution in this [gist](https://gist.github.com/vsouza/77e6b20520d07652ed7d)

