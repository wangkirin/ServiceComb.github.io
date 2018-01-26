---
title: "Basic - Develop microservice application in minutes"
lang: en
ref: quick-start-bmi-golang
permalink: /golang/quick-start-bmi/
excerpt: "Introduce how to develop a microservice application using ServiceComb in minutes through the BMI sample"
last_modified_at: 2017-09-06T00:50:00-55:00
---

{% include toc %}
## Before you begin
Walk through the instructions in the [Quick Start](/golang/quick-start/).

## Develop BMI microservice application in minutes

The workflow of BMI application have been introduced in [Quick start](/golang/quick-start/). It contains two separate microservices:

- **BMI calculator service**：provides computing services.

- **Web service**：provides both user interface and gateway service.

Now we will introduce the detailed implementation of these two microservices. The full code is on [github](https://github.com/ServiceComb-samples/go-bmi).
### Implementation of calculator
The calculator service provides capability of calculating BMI. It contains three parts:

* Business logic

* Service registration and configuration

* Start the framework

#### Business logic

Define the service structure
```go
type CalculateBmi struct {
}
```

Implement detailed calculation according to the formula \\(BMI=\frac{weight}{height^2}\\).

```go
func (c *CalculateBmi) BMIIndex(height, weight float64) (float64, error) {
	if height <= 0 || weight <= 0 {
		return 0, fmt.Errorf("Arugments must be above 0")
	}
	heightInMeter := height / 100
	bmi := weight / (heightInMeter * heightInMeter)
	return bmi, nil
}
```

Write handler function, which must be import the parameter type of `restful.Context`. `restful.Context` contains commonly used operations on Request and Response to `http` calls. Functions starts with `Read` is about read operations from Request , including `ReadEntity` , `ReadHeader` , `ReadPathParameter` , `ReadBodyParameter` , `ReadRequest` etc .; functions start with  `Writer` is  about write operation to Response , Including `WriteHeader` , `WriteJson` , `WriteError` and so on.

```go
import (
  rf "github.com/ServiceComb/go-chassis/server/restful"
)
	……
func (c *CalculateBmi) Calculate(b *rf.Context) {
	……					//Define the Response structure, omit here 
  heightStr := b.ReadQueryParameter("height")
  weightStr := b.ReadQueryParameter("weight")
  var height, weight, bmi float64
  var err error
	……  				// Convert form string to float, omit here
  if bmi, err = c.BMIIndex(height, weight); err != nil {
    errorResponse.Error = err.Error()
    b.WriteHeaderAndJson(http.StatusBadRequest, errorResponse, "application/json")
    return
  }
  result.Result = bmi
  result.CallTime = time.Now().String()
  b.WriteJson(result, "application/json")   
}
```
Specify the corresponding URL route
```go
func (c *CalculateBmi) URLPatterns() []rf.Route {
  return []rf.Route{
    {http.MethodGet, "/calculator/bmi", "Calculate"},
  }
}
```
#### Service registration and configuration

After the completion of the preparation of the business logic code, the business logic needs to be registered to the `Go-chassis` framework, and at the time of registration, the service name, ID and other attributes can be specified simultaneously.
```go
  chassis.RegisterSchema("rest",&CalculateBmi{})
```
  
In addition to some of the attributes specified in the code, more attributes are configured through the configuration file. The configuration file includes `chassis.yaml` and `microservice.yaml` , placed in the `conf` directory under the code directory. Among them, `chassis.yaml` contains public service properties, such as public AppId information,  registry type information, registry address, service protocol, transmission protocol information; `microservice.yaml`  is about the private properties of microservices, including the service name, version and so on.

    `chassis.yaml`

```yaml
APPLICATION_ID: bmi						   
cse:
  service:
    registry:
      address: http://127.0.0.1:30100    	
  protocols:
    rest:
      listenAddress: 0.0.0.0:8080           
```

​	`microservice.yaml`

```yaml
service_description:
  name: calculator						  
  version: 0.0.1 						  
```

#### Start the framework

```go
import (
  	"github.com/ServiceComb/go-chassis"
	"github.com/ServiceComb/go-chassis/core/lager"
)
	……
  if err := chassis.Init(); err != nil {           //Init the chassis framwork
    lager.Logger.Error("init failed", err)
    return
  }
  chassis.Run()                                     //Run the microservice
```

### Implementation of web-app service

The web service provides both the user interface . It contains three parts:

* Front-end static page

* Request forward

* Service configuration and startup

#### Front-end static page

Front-end static page is powered by `Bootstrap` , and through the golang official library `http.ServeFile` front-end static page will be displayed.

```go
func BmiPageHandler(w http.ResponseWriter, r *http.Request) {
  	http.ServeFile(w,r,"external/index.html")
}
```


#### Request forward

The `web-app` service forwards requests to the `calculator` service via `core.NewRestInvoker()` when it receives a request from the front-end page. In the process of forwarding the call, the user does not need to know the specific address and port of the `calculator` service. The service discovery process is automatically completed by the `go-chassis` framework.
```go
func BmiRequestHandler(w http.ResponseWriter, r *http.Request) {
  queries := r.URL.Query()
  heightStr := queries.Get("height")
  weightStr := queries.Get("weight")

  requestURI := fmt.Sprintf("cse://calculator/bmi?height=%s&weight=%s", heightStr, weightStr)
  restInvoker := core.NewRestInvoker()
  req, _ := rest.NewRequest("GET", requestURI)
  resp, _ := restInvoker.ContextDo(context.TODO(), req)

  w.Header().Set("content-type", "application/json")
  w.WriteHeader(resp.GetStatusCode())
  w.Write(resp.ReadBody())
}
```
#### Service configuration and startup

The configuration file for the `web-app` service also includes two yaml files, `chassis.yaml` and `microservice.yaml` , as follows:
`chassis.yaml`

  ```yaml
  APPLICATION_ID: bmi						
  cse:
    service:
      registry:
        address: http://127.0.0.1:30100    # ServiceCenter address
  ```

  ​	`microservice.yaml`

  ```yaml
  service_description:
    name: web-app							  
    version: 0.0.1 						  
  ```

Unlike the `calculator service` , the `web-app` is a consumer-type service within the `Go-chassis` framework, so just invoke `chassis.Init()` to initialize the `Go-chassis` framework.

```go
func main() {
  http.HandleFunc("/", BmiPageHandler)
  http.HandleFunc("/calculator/bmi", BmiRequestHandler)

  if err := chassis.Init(); err != nil {
    lager.Logger.Error("Init fail", err)
    return
  }

  port := flag.String("port", "8889", "Port web-app will listen")
  address := flag.String("address", "0.0.0.0", "Address web-app will listen")
  fullAddress := fmt.Sprintf("%s:%s", *address, *port)
  http.ListenAndServe(fullAddress, nil)
}
```








