## WS.JS
A WS-* client stack for node.js. Written in pure javascript!

**Currently supports:**

* MTOM
* WS-Security (username tokens only)
* WS-Addressing (all versions)
* HTTP(S)

For more information visit [my blog](http://webservices20.blogspot.com/).

## Install
Install with [npm](http://github.com/isaacs/npm):

    npm install ws.js

## Usage

### WS-Security (Username)
    var ws = require('ws.js')
      , Http = ws.Http
      , Security = ws.Security
      , UsernameToken = ws.UsernameToken

    var request =  '<Envelope xmlns="http://schemas.xmlsoap.org/soap/envelope/">' +
                      '<Header />' +
                        '<Body>' +
                          '<EchoString xmlns="http://tempuri.org/">' +
                            '<s>123</s>' +
                          '</EchoString>' +
                        '</Body>' +
                    '</Envelope>'

    var ctx =  { request: request 
               , url: "http://service/security"
               , action: "http://tempuri.org/EchoString"
               , contentType: "text/xml" 
               }


    var handlers =  [ new Security({}, [new UsernameToken({username: "yaron", password: "1234"})])
                    , new Http()
                    ]

    ws.send(handlers, ctx, function(ctx) {                    
      console.log("response: " + ctx.response);
    })

==>

    <Envelope xmlns="http://schemas.xmlsoap.org/soap/envelope/" xmlns:u="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-wssecurity-utility-1.0.xsd" xmlns:o="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-wssecurity-secext-1.0.xsd">
    <Header>
      <o:Security>
        <u:Timestamp>
          <u:Created>2012-02-26T11:03:40Z</u:Created>
          <u:Expires>2012-02-26T11:08:40Z</u:Expires>
        </u:Timestamp>
        <o:UsernameToken>
          <o:Username>yaron</o:Username>
          <o:Password Type="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-username-token-profile-1.0#PasswordText">1234</o:Password>
        </o:UsernameToken>
      </o:Security>
    </Header>
    <Body>
      <EchoString xmlns="http://tempuri.org/">
        <s>123</s>
      </EchoString>
    </Body>
  </Envelope>

### MTOM    
    var ws = require('ws.js')
      , Http = ws.Http
      , Mtom = ws.Mtom

    var request = '<s:Envelope xmlns:s="http://www.w3.org/2003/05/soap-envelope">' +
                    '<s:Body>' +
                      '<EchoFiles xmlns="http://tempuri.org/">' +                        
                          '<File1 />' +
                      '</EchoFiles>' +
                    '</s:Body>' +
                  '</s:Envelope>'   
    
    var ctx = { request: request
              , contentType: "application/soap+xml"
              , url: "http://localhost:7171/Service/mtom"
              , action: "http://tempuri.org/IService/EchoFiles"
              }

    //add attachment to the soap request
    ws.addAttachment(ctx, "request", "//*[local-name(.)='File1']", 
                    "me.jpg", "image/jpeg")
    
    var handlers =  [ new Mtom()
                    , new Http()
                    ];
    
    ws.send(handlers, ctx, function(ctx) {      
      //read an attachment from the soap response
      var file = ws.getAttachment(ctx, "response", "//*[local-name(.)='File1']")
      fs.writeFileSync("result.jpg", file)      
    })

==>

    --my_unique_boundary
    Content-ID: <part0>
    Content-Transfer-Encoding: 8bit
    Content-Type: application/xop+xml;charset=utf-8;type="application/soap+xml"

    <s:Envelope xmlns:s="http://www.w3.org/2003/05/soap-envelope">
      <s:Body>
        <EchoFiles xmlns="http://tempuri.org/">
          <File1>
            <xop:Include xmlns:xop="http://www.w3.org/2004/08/xop/include" href="cid:part1" />
          </File1>
        </EchoFiles>
      </s:Body>
      <s:Header />
    </s:Envelope>
    --my_unique_boundary
    Content-ID: <part1>
    Content-Transfer-Encoding: binary
    Content-Type: image/jpeg

    [binary here...]

### WS-Addressing
    var ws = require('ws.js')
      , Http = ws.Http
      , Addr = ws.Addr
      , ctx =  { request:   "<Envelope xmlns='http://schemas.xmlsoap.org/soap/envelope/'>" +
                              "<Header />" +
                                "<Body>" +
                                  "<EchoString xmlns='http://tempuri.org/'>" +
                                    "<s>123</s>" +
                                  "</EchoString>" +
                                "</Body>" +
                            "</Envelope>"

               , url: "http://localhost/service"
               , action: "http://tempuri.org/EchoString"
               , contentType: "text/xml" 
               }

    var handlers =  [ new Addr("http://schemas.xmlsoap.org/ws/2004/08/addressing")
                    , new Http()
                    ]

    ws.send(handlers, ctx, function(ctx) {                    
      console.log("response: " + ctx.response);
    })

==>

    <Envelope xmlns="http://schemas.xmlsoap.org/soap/envelope/" xmlns:ws="http://schemas.xmlsoap.org/ws/2004/08/addressing">
      <Header>
        <ws:Action>http://tempuri.org/EchoString</ws:Action>
        <ws:To>http://server/wsa/</ws:To>
        <ws:MessageID>6c4189e5-60f5-195e-b7ce-4e236d63c379</ws:MessageID>
        <ws:ReplyTo>
          <ws:Address>http://schemas.xmlsoap.org/ws/2004/08/addressing/role/anonymous</ws:Address>
        </ws:ReplyTo>
      </Header>
      <Body>
        <EchoString xmlns="http://tempuri.org/">
          <s>123</s>
        </EchoString>
      </Body>
    </Envelope>

### SSL
Just specify an http**s** address in any of the previous samples.

### All together now
    var ws = require('ws.js')
      , Http = ws.Http
      , Addr = ws.Addr
      , Mtom = ws.Mtom
      , Security = ws.Security
      , UsernameToken = ws.UsernameToken
      , ctx =  { request:   "<Envelope xmlns='http://schemas.xmlsoap.org/soap/envelope/'>" +
                              "<Header />" +
                                "<Body>" +
                                  "<EchoString xmlns='http://tempuri.org/'>" +
                                    "<File1 />" +
                                  "</EchoString>" +
                                "</Body>" +
                            "</Envelope>"

               , url: "https://localhost/service"
               , action: "http://tempuri.org/EchoString"
               , contentType: "text/xml" 
               }

    ws.addAttachment(ctx, "request", "//*[local-name(.)='File1']", 
                      "me.jpg", "image/jpeg")

    var handlers =  [ new Security({}, [new UsernameToken({username: "yaron", password: "1234"})])
                    , new Addr("http://ws-addressing/v8")
                    , new Mtom() //Mtom must be after everything except http
                    , new Http()
                    ]

    ws.send(handlers, ctx, function(ctx) {                    
      console.log("response: " + ctx.response);
    })

### More details
* [http://webservices20.blogspot.com/](http://webservices20.blogspot.com/)
* Or drop me an [email](mailto:yaronn01@gmail.com)
