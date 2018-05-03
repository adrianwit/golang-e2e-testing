## Prerequisites

1. [Download](https://github.com/viant/endly/releases) the latest secret  and endly app

2. Localhost SSH secrets

Generate pub/private key pair

```text
mkdir $HOME/.secret
ssh-keygen -b 1024 -t rsa -f id_rsa -P "" -f $HOME/.secret/id_rsa
touch ~/.ssh/authorized_keys
cat $HOME/.secret/id_rsa.pub >>  ~/.ssh/authorized_keys 
chmod u+w authorized_keys

```

Provide credentials for localhost
```text
 $  endly -c=localhost
Enter Username: 
Enter Password: 
``` 

3. Datastore secrets

Generate password for mysql
```text
 $  endly -c=mysql
Enter Username: root
Enter Password:  dev

endly -c=pg
```

4. IDE setup (ideally with YAML, JSON, CSV viewer)

a) Atom with tablr plugin  (apm install tablr)


## Typical e2e testing workflow

 1. System preparation
    - Local or remote
    - System services initialization. (RDBM, NoSQL, caching or 3rd party API, docker services)
    - Application container. (Docker, Application server, i,e, tomcat, glassfish)
 2. Application build and deployment
    -  Code checkout (local/remote).
    -  Application build (with or without docker)
    -  Application deployment (with or without docker)
 3.  Datastore preparation
    - loading schema
    - loading static data
 4. Testing 
    - HTTP runner
    - REST runner
    - Selenium runner
    - Application output verification
    - Application persisted data verification
    - Application produced log verification
    - Application with 3rd party API interaction
 5. Cleanup
    - Data cleanup
    - Application shutdown
    - Application system services shutdown


## Endly services

Service is actual execution unit, in order to run any workflow's action or piple task you have use specify : 
- service ID 
- action
- actual action request


```bash
    endly -s='*'
    endly -s='storage' -a=copy
    endly -s='exec' -a=run
```

See more about [service](https://github.com/viant/endly/tree/master/doc/service)

**Service action run:**

##### Pipeline

```bash
endly -r=hello
endly -r=hello message=greetings
```

@hello.yaml
```yaml
defaults:
  target:
    URL: ssh://127.0.0.1
    credentials: localhost
pipeline:
  hello:
    action: "exec:run"
    commands:
      - whoami
      - echo '${params.message} $cmd[0].stdout'
```

See more about [pieeline](https://github.com/viant/endly/tree/master/doc/pipeline)

##### Workflow

```bash
endly -w=hello
endly -r=hello message=greetings
```


@hello.csv

|Workflow|Name|Tasks| | |
|---|---|---|---| ---|
| |hello|%Tasks| | |
|**[]Tasks**|**Name**|**Description**|**Actions**| |
| |greet|Run greet task|%Greet| |
|**[]Greet**|**Description**|**Service**|**Action**|Request| 
| |print greeting message|exec|run|@run.yaml|

@run.yaml
```yaml
target:
  URL: ssh://127.0.0.1
  credentials: localhost
commands:
  - whoami
  - echo '${params.message} $cmd[0].stdout
```

See more about [workflow](https://github.com/viant/endly/tree/master/doc/workflow)


##### Running multiple tasks:


```bash
    endly -r=test
```

@test.yaml

```yaml
defaults:
  target:
     URL: ssh://127.0.0.1/
     credentials: localhost
pipeline:
  init:
    action: selenium:start
    version: 3.4.0
    port: 8085
    sdk: jdk
    sdkVersion: 1.8
  test:
    action: selenium:run
    browser: firefox
    remoteSelenium:
      URL: http://127.0.0.1:8085
    commands:
      - get(http://play.golang.org/?simple=1)
      - (#code).clear
      - (#code).sendKeys(package main

          import "fmt"

          func main() {
              fmt.Println("Hello Endly!")
          }
        )
      - (#run).click
      - command: output = (#output).text
        exit: $output.Text:/Endly/
        sleepTimeMs: 1000
        repeat: 10
      - close
    expect:
      output:
        Text: /Hello Endly!/
```



## E2E testing workflow generator


Open [workflow generator](https://endly-external.appspot.com/)


**Generate workflow with the following options**
1. Application
   - Template: "golang: simple web/rest"
   - Name: myapp
2. Datastore:
   - Engine: MySQL
   - Name: db1
3. Testing:
   - Selenium runner
   - REST runner
   - setup data per use case
   
4. Download

cd app:

```bash
    endly -w=manager  -t='?'
```


#### Workflow assets:



```bash
  myapp
    | - app source code
    | - endly  
    |    |- system.yaml
    |    |- datastore
    |    |    | - $db   
    |    |    |    | - dictionary
    |    |    | - schema.sql
    |    |- datastore.yaml
    |    |- app.yaml
    |    |- regression
    |    |- var
    |    |   |- init.json
    |    |- req
    |    |   |- run.yaml
    |    |- manager.csv         

```



```bash
  myapp
    | - app source code
    | - endly  
    |    |- regression
    |    |     | - regression.csv
    |    |     | - var/
    |    |     |    |- /init.json
    |    |     |    |- /test_init.json    
    |    |     | - req /
    |    |     |    | - data.yaml
    |    |     |                      
    |    |     | - use_cases
    |    |     |    | - 001_xx_case  
    |    |     |    |     | - $db_data.json
    |    |     |    |     | - selenium_test.yaml
    |    |     |    |     | - http_test.json
    |    |     |    |     | - rest_test.json    
    |    |     |    |     | - use_case.txt    
```    

1) Workflow application and datastore schema

-   [Application](https://github.com/viant/endly/tree/master/gen/template/app/go/webdb):
-   [Datastore Schema](https://github.com/viant/endly/blob/master/gen/template/datastore/mysql/ddl/schema.sql)

2) Testing workflow main assets:
- System
- Datastore
- App ([see more about build workflow](https://github.com/viant/endly/tree/master/shared/workflow/app))
- Setup data
- Regression

3) Running workflow

```bash

cd <Myapp>/endly
endly
```
  
**WEB pages**:
  - http://127.0.0.1:8080/
  - http://127.0.0.1:8080/form.html
  - http://127.0.0.1:8080/dummy.html  
  
**[REST endpoint](https://github.com/viant/endly/blob/master/gen/template/app/go/webdb/router.go)**
  - http://127.0.0.1:8080/v1/api/dummytype
  - http://127.0.0.1:8080/v1/api/dummy
  - http://127.0.0.1:8080/v1/api/dummy/1 



```bash
    
    endly -t='init,test'
    
```




4) **Adding selenium test**

- [WebDriver](https://github.com/tebeka/selenium/blob/master/selenium.go#L213)
- [WebElement](https://github.com/tebeka/selenium/blob/master/selenium.go#L370)



- **Adding new entry**

```endly
endly -r=test
```

@test.yaml
```yaml
defaults:
  target:
     URL: ssh://127.0.0.1/
     credentials: localhost
pipeline:
  init:
    action: selenium:start
    version: 3.4.0
    port: 8085
    sdk: jdk
    sdkVersion: 1.8
  test:
    action: selenium:run
    browser: firefox
    remoteSelenium:
      URL: http://127.0.0.1:8085
    commands:
      - get(http://127.0.0.1:8080/form.html)
      - (#id).clear
      - (#id).sendKeys('111111')
      - (#name).clear
      - (#name).sendKeys('dummy 123')
      - (#typeId).clear()
      - (xpath://SELECT[@id='typeId']/option[text()='type1']).click()
      - (#agree).click      
      - (#submit).click
      - command: CurrentURL = CurrentURL()
        exit: $CurrentURL:/dummy/
        sleepTimeMs: 1000
        repeat: 10
      - PageSource = pageSource()  
    expect:
      PageSource: /dummy 123/

```

- **Client side validation**

@test.yaml
```yaml
defaults:
  target:
     URL: ssh://127.0.0.1/
     credentials: localhost
pipeline:
  init:
    action: selenium:start
    version: 3.4.0
    port: 8085
    sdk: jdk
    sdkVersion: 1.8
  test:
    action: selenium:run
    browser: firefox
    remoteSelenium:
      URL: http://127.0.0.1:8085
    commands:
      - get(http://127.0.0.1:8080/form.html)
      - (#id).clear()
      - (#id).sendKeys('111111')
      - (#name).clear()
      - (#typeId).clear()
      - (xpath://SELECT[@id='typeId']/option[text()='type1']).click()
      - (#submit).click()
      - name = (xpath://DIV[preceding-sibling::INPUT[@id='name']]).text()
    expect:
      name:
        Text: /Please choose a dummy name/

```



c) As part of regression workflow

    modify regression/use_cases_001_xx_case/selenium_test.json
    
    -- modify Test{1..002} to Test{1..001} to ony work on use case 001
    -- make sure that application is up and running (endly -t='init,test'
    
    
```bash
endly -r='test'
```    



5) **Adding REST test**



a) Running a test vi shared [action workflow](https://github.com/viant/endly/blob/master/shared/workflow/action/action.csv)


```bash
endly -w=action service='rest/runner' action=send request='@send.json'
```

@send.json
```json
{

    "Method": "POST",
    "URL": "http://127.0.0.1:8080/v1/api/dummy",
    "Request":{
        "Data":{
            "TypeId":1,
            "Name":"dummy 33333"
        }
    },
	"Expect": {
        "Data": {
            "Name":"dummy 33333"
        }	
	}
}
```


b) Running as inline workflow


```bash
endly -r=test
```

test.yaml
```yaml

pipeline:
  task1:
    action: rest/runner:send
    request: "@send.json" 

```


c) As part of regression workflow

    modify regression/use_cases_001_xx_case/rest_test.json
    
    -- modify Test{1..002} to Test{1..001} to ony work on use case 001
    -- make sure that application is up and running (endly -t='init,test'
    
    
```bash
endly -r='test'
```    


6) **Adding HTTP test**

a) Running a request vi shared [action workflow](https://github.com/viant/endly/blob/master/shared/workflow/action/action.csv)


```bash
endly -w=action service='http/runner' action=send request='@send.json'
```

@send.json
```json
{
	"Requests": [
		{
			"Method": "POST",
			"URL": "http://127.0.0.1:8080/v1/api/dummy",
			"Body":"{}"
		}
	],
	"Expect": {
      "Responses":[{
          "Code":200,
          "JSONBody": {
              "Status":"error",
              "Error":"data was empty"
          }
        }]	
	}
}
```



b) Running with as inline workflow


```bash
endly -r=test
```

test.yaml
```yaml

pipeline:
  task1:
    action: http/runner:send
    request: "@send.json" 

```


c) As part of regression workflow

    modify regression/use_cases_001_xx_case/http_test.json
    
    -- modify Test{1..002} to Test{1..001} to ony work on use case 001
    -- make sure that application is up and running (endly -t='init,test'
   



7) Adding test with use case data:


a) change regression/use_cases/001_xx_case/db1_data.json to the following

```json
[
  {
    "Table": "dummy",
    "Value": {
      "id": "$seq.dummy",
      "name": "My super dummy 1",
      "type_id": 2
    },
    "PostIncrement": [
      "seq.dummy"
    ],
    "Key": "${tagId}_dummy"
  }
]
```

b) note that in the regression/var/test_init.json the added record is placed to the context state:

so that it can be reference as $dummy.<FIELD>

```json
[
  {
    "Name": "dummy",
    "From": "<-dsunit.${tagId}_dummy"
  }
]
```


```bash
endly -t=test
```


c) modify regression/use_cases_001_xx_case/rest_test.json

@send.json
```json
{

    "Method": "GET",
    "URL": "http://127.0.0.1:8080/v1/api/dummy/${dummy.id}",
    "Request":{},
	"Expect": {
        "Data": {
            "Id":"$dummy.id",
            "Name":"$dummy.name",
            "TypeId":"${dummy.type_id}"
        }	
	}
}
```




## Validation


clone this project to run the following examples:


```bash
endly -s=validator -a=assert

endly -r=run
```

@run.yaml
```yaml
pipeline:
  build:
    action: workflow:print
    message: building app ...
  test:
    workflow: assert
    actual:
      key1: value1
      key2: value20
      key3: value30
    expected:
      key1: value1
      key2: value2
      key3: value3
      key4: value4     
```


```text
endly -w=assert -t='?'
endly -w assert -p -f=yaml
```

**Simple validation**
```text
endly -w=assert actual A expected B
``` 

**Data structure validation**
```text
endly -w=assert actual '@actual1.json' expected '@expected1.json'
``` 

**Data structure transformation validation**
```text
endly -d=true -w=assert actual '@actual2.json' expected '@expected2.json'
```

**Data structure alternative transformation validation (switch/case)**
```text
endly -d=true -w=assert actual '@actual3.json' expected '@expected3.json'
```

See more about [validation expression](https://github.com/viant/assertly#validation)

