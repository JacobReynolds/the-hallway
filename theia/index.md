---
title: theia
layout: default
description: greek goddess or primordial planet?
nav_order: 9
---

# 👀 theia

The [Greek Goddess of Sight](https://en.wikipedia.org/wiki/Theia), or alternatively, the primordial planet that [smashed into Earth to create the Moon](<https://en.wikipedia.org/wiki/Theia_(planet)>). Theia was an idea I had to built out a sensor network of honeypots to be able to report on emerging trends in internet exploitation. You can think of it as being very similar to [Greynoise](https://www.greynoise.io/).

## Please help me

My assumption was that ChatGPT would be a massive help here. I wanted to look at a lot of non-http protocols, things like SMB, SMTP, etc... and see what people were throwing at them. Given I know next to nothing about SMB, I figured I could convince Jippity to write a handler for me. As expected, when asking it to write a golang server that'll accept SMB, I get:

> Implementing the full Server Message Block (SMB) protocol handshake in a simple Go program is a complex and extensive task due to the SMB protocol's complexity and the security implications of handling such connections.

and then it spits out a bunch of golang functions with `// Implement parsing logic here.` comments above each of them.

Before we get ahead of ourselves though, let's try to just get SSH working.

```go
package main

import (
	"log"
	"strings"

	"github.com/gliderlabs/ssh"
)

func main() {
	err := ssh.ListenAndServe(":22", nil, ssh.PasswordAuth(func(ctx ssh.Context, pass string) bool {
		ctx.User()
		ip := strings.Split(ctx.RemoteAddr().String(), ":")[0]
		username := ctx.User()
		SendSensorData(ip, username, pass)
		return false
	}))

	if err != nil {
		log.Fatal(err)
	}

}
```

That was pretty easy, spin up a server on 22, enable password authentication, and we can now start logging the dictionaries people use to try and bruteforce SSH passwords! A little fun fact for the attentive reader, I'm writing this about 4 months after I shut down the project so I don't have access to the data anymore sadly. But the password results were basically what you'd expect, lots of `admin:admin` and `1234:1234` were submitted. I then stored all of these analytics in a [Clickhouse](https://clickhouse.com/) database, because I expected to scale quite significantly and only needed to perform [OLAP](https://en.wikipedia.org/wiki/Online_analytical_processing) queries on it.

I was then quickly able to add the same functionality for the following:

### SMTP login detection

```go
package main

import (
	"bufio"
	"fmt"
	"log"
	"net"
	"strings"
)

func main() {
	listener, err := net.Listen("tcp", "0.0.0.0:25")
	if err != nil {
		log.Fatal(err)
	}
	defer listener.Close()

	fmt.Println("SMTP honeypot is listening on port 25...")

	for {
		conn, err := listener.Accept()
		if err != nil {
			log.Println(err)
			continue
		}

		go handleConnection(conn)
	}
}

func handleConnection(conn net.Conn) {
	defer conn.Close()

	clientIP := conn.RemoteAddr().(*net.TCPAddr).IP.String()
	fmt.Printf("New connection from %s\n", clientIP)

	conn.Write([]byte("220 SMTP hello\r\n"))

	requests := []string{}

	scanner := bufio.NewScanner(conn)
	for scanner.Scan() {
		command := scanner.Text()
		fmt.Printf("Received from %s: %s\n", clientIP, command)
		requests = append(requests, command)

		if strings.HasPrefix(command, "QUIT") {
			conn.Write([]byte("221 Bye\r\n"))
			break
		}

		conn.Write([]byte("250 OK\r\n"))
	}

	if scanner.Err() != nil {
		log.Printf("Error reading from %s: %s", clientIP, scanner.Err())
	}

	SendSensorData(clientIP, strings.Join(requests, "\n"))
}
```

### Port scanning detection

```go
package main

import (
	"fmt"
	"net"
	"os"
	"strconv"
	"strings"
	"sync"
)

var wg sync.WaitGroup

func main() {
	PORT_NUMBERS := []int{21, 23, 53, 110, 111, 135, 139, 143, 443, 445, 993, 995, 1723, 3306, 3389, 5900, 8080}

	for _, port := range PORT_NUMBERS {
		wg.Add(1)
		go listenOnPort(port)
	}
	wg.Wait()

	fmt.Println("Exiting")
}

// https://coderwall.com/p/wohavg/creating-a-simple-tcp-server-in-go
func listenOnPort(port int) {
	defer wg.Done()
	l, err := net.Listen("tcp", "0.0.0.0:"+strconv.Itoa(port))
	if err != nil {
		fmt.Println("Error listening:", err.Error())
		return
	}
	defer l.Close()
	fmt.Println("Listening on " + "0.0.0.0:" + strconv.Itoa(port))
	for {
		conn, err := l.Accept()
		if err != nil {
			fmt.Println("Error accepting: ", err.Error())
			os.Exit(1)
		}
		go handleRequest(conn, port)
	}
}

// Handles incoming requests.
func handleRequest(conn net.Conn, port int) {
	defer conn.Close()
	ipAddress := strings.Split(conn.RemoteAddr().String(), ":")[0]
	SendSensorData(ipAddress, port)
}
```

### HTTP requests

```go
package main

import (
	"encoding/json"

	"github.com/gofiber/fiber/v2"
)

func main() {
	app := fiber.New(fiber.Config{})

	app.Get("*", func(c *fiber.Ctx) error {
		path := c.Path()
		ip := c.IP()

		sensorData, _ := json.Marshal(map[string]string{
			"ip":   ip,
			"path": path,
		})
		SendSensorData(sensorData)

		return c.SendStatus(200)
	})

	app.Listen(":80")
}

```

## Data feeds

I also wanted to add some data feeds so I could provide IP address information, which came from the lovely [ipinfo.io](https://ipinfo.io). The other piece of data I thought was cool was aggregating all TOR exit nodes, and being able to see if any of our traffic was coming from them. Easily enough, that's available [here](https://check.torproject.org/torbulkexitlist). That was quick! I did end up getting a few hits from TOR exit nodes, primarily for web traffic if I'm remembering correctly, so that was pretty nifty.

## Setting Sail

I deployed this on AWS across ~15 regions and let it run for 3 months or so. Unfortunately work got busy and I didn't have time to invest into the project and decided to shut it off to save my cloud computing bill :/. I still think this would be an awesome project, however the people over at Greynoise are doing a killer job, so it'd be extremely tough competition.

As always, another anticlimactic ending for ya, but theia was too cool of a name not to write about.
