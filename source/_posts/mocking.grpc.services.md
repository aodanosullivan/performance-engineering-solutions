---
title: Mocking gRPC services
date: 2023-05-27T21:00:00+01:00
author: Piotr
photos: 
	- /img/posts/grpc.jpg
tags: [mocks,grpc]
categories: 
  - [mocks]
---


gRPC is increasing in popularity. Because of this, there may be a need to mock gRPC based services in early development and testing. This article shows current possibilities, as of mid of 2023.

<!--more-->

In general we can divide existing software into two group - free and paid software. Obviously we are interested more in free option, so here we are looking more into that one.

### Paid solutions

To keep all options available, it can be mentioned, at least one paid application that provides possibility of gRPC mocking. 

[Traffic Parrot](https://trafficparrot.com/), as stated on their web pages _provides API and service simulation, service virtualization and API mocking software_. Evaluation version supports wide range of protocols already, including gRPC.

There is also a possibility to request dedicated support for many others, as stated on their [web](https://trafficparrot.com/request_features.html)

As we are interested in gRPC only here are links, that may be helpful:
Resources:

 * [documentation](https://trafficparrot.com/documentation/5.39.x/grpc.html)
 * [demo](https://www.youtube.com/watch?v=cDCIj-ql83s)

Watching demo may be also a good idea for general overview, how most of mocks (also the free ones) work in general.

### Free solutions

As there are free-of-charge solutions in the internet, let's discuss them briefly, without getting into details too much.

### Wiremock

[Wiremock](https://wiremock.org/) is java-based solution, that provides easily configurable stand-alone jar that can serve as a mock. Code repo is [here](https://github.com/wiremock/wiremock)
Wiremock as of now, does not support gRPC natively. It's planned to be included in the [future](https://github.com/orgs/wiremock/projects/5/views/1?filterQuery=grpc&pane=issue&itemId=26295573), but without any hard dates or promises. Still, there is open source solution based on wiremock, called [grpc-wiremock](https://github.com/Adven27/grpc-wiremock), that can be used.

This solution is dedicated to be run with docker. With some effort it's also possible to run it locally.
Proposed docker file example is provided within [repo](https://github.com/Adven27/grpc-wiremock/blob/master/example) together with demo proto files and wiremock configuration. There is also .json file for testing with ghz binary available from this [source](https://github.com/bojand/ghz/releases). Setup is quite straightforward.

Example wallet service will be used for a quick demo.

Structure used for testing (with default content):

{% codeblock %}
    proto/
    ├── common.proto
    └── wallet.proto
    
    __files/
    └── no-balance.json
    
    mappings/
    ├── createTransaction.json
    ├── getUserBalance.json
    └── searchTransaction.json
{% endcodeblock %}

All mentioned directories are copied into Docker image, as proposed by example [Dockerfile](https://github.com/Adven27/grpc-wiremock/blob/master/example/Dockerfile).

Docker image is started, and last part should notify about server status, like:

{% codeblock %}
    <timestamp>  INFO 146 --- [           main] io.adven.grpc.wiremock.HttpMock          : WireMock server is started:
    port:                         8888
    enable-browser-proxying:      false
    disable-banner:               false
    no-request-journal:           false
    verbose:                      false
    
    <timestamp>  INFO 146 --- [           main] io.adven.grpc.wiremock.GrpcWiremock      : gRPC server is started: ServerImpl{logId=2, transportServer=NettyServer{logId=1, addresses=[0.0.0.0/0.0.0.0:50000]}}
    Registered services:
     * grpc.reflection.v1alpha.ServerReflection
     * api.wallet.WalletService
     * api.wallet.BalanceService
{% endcodeblock %}

As server supports gRPC reflection, grpcurl can be used to list available services:

{% codeblock lang:bash %}
    $ grpcurl -plaintext 127.0.0.1:50000 list
    api.wallet.BalanceService
    api.wallet.WalletService
    grpc.reflection.v1alpha.ServerReflection
{% endcodeblock %}

Then smoke test for BalanceSerice be done with grpcurl utility

{% codeblock lang:bash %}
    grpcurl \
     -plaintext \
     -d '{"user_id": 1, "currency": "EUR"}' \
     localhost:50000 \
     api.wallet.BalanceService/getUserBalance
{% endcodeblock %}

Server should return:

{% codeblock lang:json %}
    {
      "balance": {
        "amount": {
          "value": {
            "decimal": "0.0"
          },
          "value_present": true
        },
        "currency": {
          "value": "ANY"
        }
      }
    }
{% endcodeblock %}

This service was successfully mocked with example provided by mock developer. For a reference same service will be used to test other mocks.

### Mountebank

[Mountebank](http://www.mbtest.org/) is another great tool, that is used for mocking. Code can be found [here](https://github.com/bbyars/mountebank). In a similar way as wiremock, it doesn't support gRPC by default, but it's possible to install plugin, that will make it working.

This can be done via npm:

{% codeblock lang:bash %}
    npm install -g mountebank-grpc-mts
{% endcodeblock %}

Then gRPC support should be added into configuration:

{% codeblock lang:bash %}
    cd ${mountebankBaseDir}
    echo '{ "grpc": { "createCommand": "mb-grpc" } }' > protocols-grpc.json
{% endcodeblock %}

`${mountebankBaseDir}` can point to any existing directory of your choice.

After starting server with command

{% codeblock lang:bash %}
    $ mb start --loglevel debug --protofile protocols-grpc.json
{% endcodeblock %}

You should be able see information like:

{% codeblock %}
    info: [mb:2525] Loaded custom protocol grpc
    info: [mb:2525] mountebank v2.8.2 now taking orders - point your browser to http://localhost:2525/ for help
    debug: [mb:2525] config: {"options":{"protofile":"protocols-grpc.json","port":2525,"noParse":false,"no-parse":false,"formatter":"mountebank-formatters","pidfile":"mb.pid","allowInjection":false,"allow-injection":false,"localOnly":false,"local-only":false,"ipWhitelist":["*"],"ip-whitelist":"*","mock":false,"debug":false,"origin":false,"apikey":null,"log":{"level":"debug","transports":{"console":{"colorize":true,"format":"%level: %message"},"file":{"path":"mb.log","format":"json"}}}},"process":{"nodeVersion":"v18.16.0","architecture":"x64","platform":"linux"}}
{% endcodeblock %}

The structure in this case is slightly different than for wiremock:

{% codeblock %}
    proto
    ├── common.proto
    └── wallet.proto
    
    imposters
    └── grpc.json
    
    stubs
    └── wallet.balance.json
{% endcodeblock %}

Content of `proto` directory is same as for wiremock and it is located in `/tmp/mountebank/grpc/proto` directory. `imposters` contains gRPC services definition:

{% codeblock lang:json %}
    {
      "protocol": "grpc",
      "port": 4545,
      "loglevel": "debug",
      "recordRequests": true,
      "_note_services": "need the name of the package, service and protofile location for this to load",
      "services": {
         "api.wallet.BalanceService": { "file": "wallet.proto" },
         "api.wallet.WalletService": { "file": "wallet.proto" }
    
      },
      "options": {
        "protobufjs": {
          "_note": "any options to protobufjs",
          "includeDirs": ["/tmp/mountebank/grpc/proto"]
        }
      }
    }
{% endcodeblock %}

Important thing here is to point server to proto directory, where services and dependencies are defined.
In stubs, there are responses definition in json format. In this case the content is:

{% codeblock lang:json %}
    {
      "stub": {
        "predicates": [
          { "equals": { "path": "/api.wallet.BalanceService/getUserBalance" }, "caseSensitive": false }
        ],
        "responses": [
          {
            "is": {
              "value": 
                {
                  "balance": {
                    "amount": {
                      "value": {
                        "decimal": "0.0"
                      },
                      "value_present": true
                    },
                    "currency": {
                      "value": "ANY",
                      "value_present": false
                    }
                  }
                }
            }
          }
        ]
      }
    }
{% endcodeblock %}

To make mock working imposter and stub should be applied to the server.

Imposter import:

{% codeblock lang:bash %}
    $ curl -X POST  -d@/tmp/import/grpc.json  http://127.0.0.1:2525/imposters
    {
      "protocol": "grpc",
      "port": 4545,
      "numberOfRequests": 0,
      "recordRequests": true,
      "requests": [],
      "stubs": [],
      "_links": {
        "self": {
          "href": "http://127.0.0.1:2525/imposters/4545"
        },
        "stubs": {
          "href": "http://127.0.0.1:2525/imposters/4545/stubs"
        }
      }
    }
{% endcodeblock %}

This is also reported on server side:

{% codeblock %}
    info: [mb:2525] POST /imposters
    debug: [mb:2525] ::ffff:127.0.0.1:56886 => {"protocol":"grpc","port":4545,"loglevel":"debug","recordRequests":true,"_note_services":"need the name of the package, service and protofile location for this to load","services":{"api.wallet.BalanceService":{"file":"wallet.proto"},"api.wallet.WalletService":{"file":"wallet.proto"}},"options":{"protobufjs":{"_note":"any options to protobufjs","includeDirs":["/tmp/mountebank/grpc/proto"]}}}
    debug: [grpc:4545] {"port":4545,"encoding":"utf8","services":{"api.wallet.BalanceService":{"file":"wallet.proto"},"api.wallet.WalletService":{"file":"wallet.proto"}}}
    info: [grpc:4545] Open for business...
    info: [grpc:4545] server started on port '4545'
{% endcodeblock %}

Similar thing should be done for stub:

{% codeblock lang:bash %}
    $ curl  -X POST  -d@/tmp/import/wallet.balance.json  http://127.0.0.1:2525/imposters/4545/stubs
    {
      "protocol": "grpc",
      "port": 4545,
      "numberOfRequests": 0,
      "recordRequests": true,
      "requests": [],
      "stubs": [
        {
          "predicates": [
            {
              "equals": {
                "path": "/api.wallet.BalanceService/getUserBalance"
              },
              "caseSensitive": false
            }
          ],
          "responses": [
            {
              "is": {
                "value": {
                  "balance": {
                    "amount": {
                      "value": {
                        "decimal": "0.0"
                      },
                      "value_present": true
                    },
                    "currency": {
                      "value": "ANY",
                      "value_present": false
                    }
                  }
                }
              }
            }
          ],
          "_links": {
            "self": {
              "href": "http://127.0.0.1:2525/imposters/4545/stubs/0"
            }
          }
        }
      ],
      "_links": {
        "self": {
          "href": "http://127.0.0.1:2525/imposters/4545"
        },
        "stubs": {
          "href": "http://127.0.0.1:2525/imposters/4545/stubs"
        }
      }
    }
{% endcodeblock %}

Triggered server message looks like:

{% codeblock %}
    info: [mb:2525] POST /imposters/4545/stubs
{% endcodeblock %}

Now server is expected to be configured. Let's check same example as for wiremock

{% codeblock lang:bash %}
    grpcurl -plaintext -import-path /tmp/mountebank/grpc/proto -proto wallet.proto -d '{"user_id": 1, "currency": "EUR"}' 127.0.0.1:4545 api.wallet.BalanceService/getUserBalance
    {
      "balance": {
        "amount": {
          "value": {
            "decimal": "0.0"
          },
          "valuePresent": true
        },
        "currency": {
          "value": "ANY"
        }
      }
    }
{% endcodeblock %}

Response is successful, and in debug server messages can be seen how request was processed:

{% codeblock %}
    debug: [grpc:4545] sending unary-unary rpc
    debug: [grpc:4545] send request to mountebank
    debug: [grpc:4545] url='http://localhost:2525/imposters/4545/_requests', data='{"request":{"peer":"127.0.0.1:52350","path":"/api.wallet.BalanceService/getUserBalance","value":{"user_id":"1","currency":"EUR"},"metadata":{"initial":{"user-agent":"grpcurl/v1.8.7 grpc-go/1.48.0"}}}}'
    debug: [mb:2525] POST /imposters/4545/_requests
    debug: [grpc:4545] using predicate match: [{"equals":{"path":"/api.wallet.BalanceService/getUserBalance"},"caseSensitive":false}]
    debug: [grpc:4545] generating response from {"is":{"value":{"balance":{"amount":{"value":{"decimal":"0.0"},"value_present":true},"currency":{"value":"ANY","value_present":false}}}}}
    debug: [grpc:4545] response.data="{"response":{"value":{"balance":{"amount":{"value":{"decimal":"0.0"},"value_present":true},"currency":{"value":"ANY","value_present":false}}}}}"
{% endcodeblock %}

In this way it was showed how mountebank can be used to work with gRPC.

### Camouflage

Now we can take a look on the only mock server, that offer gRPC support "by default".
[Camouflage](https://testinggospels.github.io/camouflage/) is npm-installable application. Code is available in [github repo](https://github.com/testinggospels/camouflage). Again, we re-use protofiles from previous examples to check how mocking can be implemented.

After installing camouflage it should be initiated:

{% codeblock lang:bash %}
    export camouflageBaseDir=/tmp/camouflage
    mkdir -p ${camouflageBaseDir} \
     && cd ${camouflageBaseDir} \
     && camouflage init
{% endcodeblock %}

Structure should be created:

{% codeblock lang:bash %}
    $ ls -1 /tmp/camouflage/
    certs
    config.yml
    custom_handlebar.json
    grpc
    mocks
    plconfig.js
    thrift
    ws_mocks
    
    ls -1 /tmp/camouflage/grpc/
    mocks
    protos
{% endcodeblock %}

It goes with predefined grpc set for protos and mocks. It will be removed and replaced with set related to BalanceService.

{% codeblock lang:bash %}
    rm -rf /tmp/camouflage/grpc/mocks /tmp/camouflage/grpc/protos /tmp/camouflage/.protoignore
{% endcodeblock %}

Prepared structure to be used:

{% codeblock %}
    /tmp/import/proto
    ├── common.proto
    └── wallet.proto
    
    /tmp/import/mocks/
    └── api
        └── wallet
            └── BalanceService
                └── getUserBalance.mock
{% endcodeblock %}

Proto files are same as previously and in `mocks` directory file is created to mimic same replay as seen before:

{% codeblock lang:bash %}
    $ cat /tmp/import/mocks/api/wallet/BalanceService/getUserBalance.mock 
    {
      "balance": {
        "amount": {
          "value": {
            "decimal": "0.0"
          },
          "value_present": true
        },
        "currency": {
          "value": "ANY",
          "value_present": false
        }
      }
    }
{% endcodeblock %}

Prepared data should be placed in created structures:

{% codeblock lang:bash %}
    $ cp -a /tmp/import/* /tmp/camouflage/grpc
{% endcodeblock %}

Mock configuration should be align to our changes, to enable gRPC.
You need to edit `/tmp/camouflage/config.yml` file.
Backup of original file is backed up.

{% codeblock lang:bash %}
    $ cp /tmp/camouflage/config.yml /tmp/camouflage/config.yml.old 
{% endcodeblock %}

Changes are done:

 * loglevel is set to debug
 * http.enable is set to false
 * grpc.enable is set to true
 * name of proto dir is updates from `protos` to `proto`

Changes can be verified with diff tool:

{% codeblock lang:bash %}
    diff /tmp/camouflage/config.yml /tmp/camouflage/config.yml.old
    1c1
    < loglevel: debug
    ---
    > loglevel: info
    11c11
    <     enable: false
    ---
    >     enable: true
    25c25
    <     enable: true
    ---
    >     enable: false
    29c29
    <     protos_dir: "./grpc/proto"
    ---
    >     protos_dir: "./grpc/protos"
{% endcodeblock %}

Then server is ready and can be started

{% codeblock lang:bash %}
    cd /tmp/camouflage/
    camouflage --config config.yml
{% endcodeblock %}

Following information from server logs is related with gRPC and provided setup:

{% codeblock %}
    <timestamp> debug: Found protofile: /tmp/camouflage/grpc/proto/common.proto 
    <timestamp> debug: Found protofile: /tmp/camouflage/grpc/proto/wallet.proto 
    <timestamp> debug: Using proto-loader config as: {"keepCase":true,"includeDirs":["./grpc/protos"]} 
    <timestamp> debug: Ignoring protofiles:  
    <timestamp> debug: Using insecure gRPC server credentials. 
    <timestamp> debug: Registering Unary method: createTransaction 
    <timestamp> debug: Registering method with server side streaming: searchTransaction 
    <timestamp> debug: Registering Unary method: getUserBalance 
    (node:1211) Warning: common.proto not found in any of the include paths ./grpc/protos
    <timestamp> info: Worker sharing gRPC server at 0.0.0.0:4312 ⛳ 
{% endcodeblock %}

Test is done in a same way as before:

{% codeblock lang:bash %}
    grpcurl -plaintext -import-path /tmp/camouflage/grpc/proto -proto wallet.proto -d '{"user_id": 1, "currency": "EUR"}' 127.0.0.1:4312 api.wallet.BalanceService/getUserBalance
    {
      "balance": {
        "amount": {
          "value": {
            "decimal": "0.0"
          },
          "valuePresent": true
        },
        "currency": {
          "value": "ANY"
        }
      }
    }
{% endcodeblock %}

And reported on server side:

{% codeblock %}
    <timestamp> debug: Unary Request: {"user_id":{"low":1,"high":0,"unsigned":true},"currency":"EUR"}. Metadata: {"user-agent":["grpcurl/v1.8.7 grpc-go/1.48.0"]} 
    <timestamp> debug: Mock file path: grpc/mocks/api/wallet/BalanceService/getUserBalance.mock 
    <timestamp> debug: Response: {
      "balance": {
        "amount": {
          "value": {
            "decimal": "0.0"
          },
          "value_present": true
        },
        "currency": {
          "value": "ANY",
          "value_present": false
        }
      }
    }
{% endcodeblock %}

In this time mocking was successful again.

### Comparison

Table below shows a brief summary for all discussed mocks.


| item               | grpc-wiremock | mountebank      | camouflage             |
|--------------------|---------------|-----------------|------------------------|
| code               | Java          | JavaScript, EJS | TypeScript, JavaScript |
| default setup      | docker image  | npm             | npm                    |
| gRPC support type  | custom repo   | npm plugin      | built-in               |
| gRPC reflection    | yes           | no              | no                     |

