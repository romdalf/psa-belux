# Podman-Desktop Demo
This piece of software and container creation has been used during to
the Red Hat Partner Meetup for the BeLux region on June 6th 2023.

## The Hello app 

```go
package main

import (
    "fmt"
	"log"
    "net/http"
)

func main() {

	// print the hello message with the URL path 
	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        fmt.Fprintf(w, "Hello from URL path: %s\n", r.URL.Path)

		// if URL path is root - propose a test
		if r.URL.Path == "/" {
			fmt.Fprintf(w, "Try to add /partner as a path.")
		}

		// print the URL path at the console
		if r.URL.Path != "/favicon.ico" {
			fmt.Printf("User requested the URL path: %s\n", r.URL.Path)
		}
    })

	// print message at the console
	fmt.Println("Red Hat Partner Meetup BeLux - Hello World")
	fmt.Println("--> Server running on http://localhost:8080")

	// start the service and listen on the given port
    if err := http.ListenAndServe(":8080", nil); err != nil {
		// print error messages at the console
		log.Fatal(err)
	}
}
```

## Kubernetes Pod Definition

```YAML
apiVersion: v1
kind: Pod
metadata:
  annotations:
    io.podman.annotations.ulimit: nofile=524288:524288,nproc=7252:7252
  labels:
    app: hello-app
  name: hello-app-pod
spec:
  containers:
  - image: ghcr.io/romdalf/psa-belux/hello-path:0.1
    name: hello
    ports:
    - containerPort: 8080
      hostPort: 8080
    tty: true
```

## Kubernetes Service Definition

```YAML
apiVersion: v1
kind: Service
metadata:
  name: hello-pod-8080
  namespace: default
spec:
  clusterIP: 10.96.135.26
  clusterIPs:
  - 10.96.135.26
  internalTrafficPolicy: Cluster
  ipFamilies:
  - IPv4
  ipFamilyPolicy: SingleStack
  ports:
  - name: hello-pod-8080
    port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    app: hello-app
  sessionAffinity: None
  type: ClusterIP
status:
  loadBalancer: {}
```
