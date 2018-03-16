<img src="https://raw.githubusercontent.com/nordic-institute/X-Road-code-samples/master/img/x-road_logo.png" width="400" title="X-Road Logo">

# X-Road Code Samples

This repository provides X-Road code samples that show how to consume services
through X-Road. The examples show how X-Road compatible clients can be built
using different technologies and they help you to get started quickly. Currently
there are examples available in Java and JavaScript.

You can get started with development and testing without your own Security Server
installation using [Example Adapter](https://github.com/vrk-kpa/xrd4j/tree/master/example-adapter)
service. Another alternative for development/testing is [X-Road Test Service](https://github.com/petkivim/x-road-test-service). All the examples in this repository use Example Adapter service.

**List of Reusable X-Road Components**

In addition, this repository also provides a [list](COMPONENTS.md) of reusable X-Road components implemented by the X-Road community. Before adding new components to the list please take a look at the [Component Description Guidelines](COMPONENT_DESCRIPTION_GUIDELINES.md).

## Prerequisites

Setup [Example Adapter](https://github.com/vrk-kpa/xrd4j/tree/master/example-adapter) service:

* Clone the repository, compile the XRd4J library, compile Example Adapter, start Example Adapter.

OR

* Use Example Adapter Docker [image](https://hub.docker.com/r/niis/example-adapter/).

```
docker run -p 8080:8080 niis/example-adapter
```

Once started the Example Adapter service
can be accessed at http://localhost:8080/example-adapter-x.x.x-SNAPSHOT/Endpoint where **x.x.x needs to be replaced with the current version number**.

## Java

### XRd4J

[XRd4J](https://github.com/vrk-kpa/xrd4j) is a Java library for building X-Road v6 Adapter Servers and clients.

This example demonstrates how to invoke ```helloService``` service with one request parameter.

```
import fi.vrk.xrd4j.client.SOAPClient;
import fi.vrk.xrd4j.client.SOAPClientImpl;
import fi.vrk.xrd4j.client.deserializer.ServiceResponseDeserializer;
import fi.vrk.xrd4j.client.serializer.ServiceRequestSerializer;
import fi.vrk.xrd4j.common.member.ConsumerMember;
import fi.vrk.xrd4j.common.member.ProducerMember;
import fi.vrk.xrd4j.common.message.ServiceRequest;
import fi.vrk.xrd4j.common.message.ServiceResponse;
import fi.vrk.xrd4j.common.util.MessageHelper;
import fi.vrk.xrd4j.common.util.SOAPHelper;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
...
...
String url = "http://localhost:8080/example-adapter-x.x.x-SNAPSHOT/Endpoint";

// Consumer that is calling a service
ConsumerMember consumer = new ConsumerMember("NIIS-TEST", "GOV", "1234567-8", "TestSystem");

// Producer providing the service
ProducerMember producer = new ProducerMember("NIIS-TEST", "GOV", "9876543-1", "DemoService", "helloService", "v1");
producer.setNamespacePrefix("ts");
producer.setNamespaceUrl("http://test.x-road.fi/producer");

// Create a new ServiceRequest object, unique message id is generated by MessageHelper.
// Type of the ServiceRequest is the type of the request data (String in this case)
ServiceRequest<String> request = new ServiceRequest<String>(consumer, producer, MessageHelper.generateId());

// Set username
request.setUserId("jdoe");

// Set message id
request.setId("12345");

// Set request data
request.setRequestData("Test message");

// Application specific class that serializes request data
ServiceRequestSerializer serializer = new HelloServiceRequestSerializer();

// Application specific class that deserializes response data
ServiceResponseDeserializer deserializer = new HelloServiceResponseDeserializer();

// Create a new SOAP client
SOAPClient client = new SOAPClientImpl();

// Send the ServiceRequest, result is returned as ServiceResponse object
ServiceResponse<String, String> serviceResponse = client.send(request, url, serializer, deserializer);

// Print out the SOAP message received as response
logger.info(SOAPHelper.toString(serviceResponse.getSoapMessage()));

// Print out only response data. In this case response data is a String.
logger.info(serviceResponse.getResponseData());

}

public class HelloServiceRequestSerializer extends AbstractServiceRequestSerializer {

  @Override
  protected void serializeRequest(ServiceRequest request, SOAPElement soapRequest, SOAPEnvelope envelope) throws SOAPException {
      SOAPElement data = soapRequest.addChildElement(envelope.createName("name"));
      data.addTextNode((String) request.getRequestData());
  }
}

public class HelloServiceResponseDeserializer extends AbstractResponseDeserializer<String, String> {
  @Override
  protected String deserializeRequestData(Node requestNode) throws SOAPException {
      for (int i = 0; i < requestNode.getChildNodes().getLength(); i++) {
          if (requestNode.getChildNodes().item(i).getLocalName().equals("name")) {
              return requestNode.getChildNodes().item(i).getTextContent();
          }
      }
      return null;
    }

  @Override
  protected String deserializeResponseData(Node responseNode, SOAPMessage message) throws SOAPException {
      for (int i = 0; i < responseNode.getChildNodes().getLength(); i++) {
          if (responseNode.getChildNodes().item(i).getLocalName().equals("message")) {
              return responseNode.getChildNodes().item(i).getTextContent();
          }
      }
      return null;
  }
}
```

Sample code is available at https://github.com/nordic-institute/X-Road-code-samples/tree/master/full-samples/xrd4j.

### wsimport

[Wsimport](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/wsimport.html) is a Java tool
for generating the required Java artifacts for invoking a SOAP service. Wsimport
uses the WSDL description of the service for artifact generation.

This example demonstrates how to use wsimport tool for generating the client
code and invoking Example Adapter ```helloService``` service with one request
parameter.

```
TestService service = new TestService();
TestServicePortType port = service.getTestServicePort();
WSBindingProvider bp = (WSBindingProvider) port;

SOAPFactory factory = SOAPFactory.newInstance();

SOAPElement clientHeader = factory.createElement(new QName("http://x-road.eu/xsd/xroad.xsd", "client"));
clientHeader.addNamespaceDeclaration("id", "http://x-road.eu/xsd/identifiers");
clientHeader.addAttribute(new QName("", "objectType", "id"), "SUBSYSTEM");

SOAPElement xRoadInstance = clientHeader.addChildElement("xRoadInstance", "id");
xRoadInstance.addTextNode("NIIS-TEST");

SOAPElement memberClass = clientHeader.addChildElement("memberClass", "id");
memberClass.addTextNode("GOV");

SOAPElement memberCode = clientHeader.addChildElement("memberCode", "id");
memberCode.addTextNode("1234567-8");

SOAPElement subsystem = clientHeader.addChildElement("subsystemCode", "id");
subsystem.addTextNode("TestClient");

// Service soap header
SOAPElement serviceHeader = factory.createElement(new QName("http://x-road.eu/xsd/xroad.xsd", "service"));
serviceHeader.addNamespaceDeclaration("id", "http://x-road.eu/xsd/identifiers");

serviceHeader.addAttribute(new QName("", "objectType", "id"), "SERVICE");

xRoadInstance = serviceHeader.addChildElement("xRoadInstance", "id");
xRoadInstance.addTextNode("NIIS-TEST");

memberClass = serviceHeader.addChildElement("memberClass", "id");
memberClass.addTextNode("GOV");

memberCode = serviceHeader.addChildElement("memberCode", "id");
memberCode.addTextNode("1234567-8");

subsystem = serviceHeader.addChildElement("subsystemCode", "id");
subsystem.addTextNode("DemoService");

SOAPElement serviceCode = serviceHeader.addChildElement("serviceCode", "id");
serviceCode.addTextNode("helloService");

SOAPElement serviceVersion = serviceHeader.addChildElement("serviceVersion", "id");
serviceVersion.addTextNode("v1");

bp.setOutboundHeaders(
    Headers.create(clientHeader),
    Headers.create(serviceHeader),
    Headers.create(new QName("http://x-road.eu/xsd/xroad.xsd", "id"), "12345"),
    Headers.create(new QName("http://x-road.eu/xsd/xroad.xsd", "userId"), "jdoe"),
    Headers.create(new QName("http://x-road.eu/xsd/xroad.xsd", "protocolVersion"), "4.0")
);
// Call helloService
logger.info(port.helloService("Test User"));
```

Sample code is available at https://github.com/nordic-institute/X-Road-code-samples/tree/master/full-samples/wsimport.

### jax-ws

Java API for XML Web Services (JAX-WS) is a technology for building web services and clients that communicate using XML.

This example demonstrates how to invoke ```helloService``` service with one request parameter.

```
QName SERVICE_NAME = new QName("http://test.x-road.fi/producer", "testService");
QName SERVICE_PORT = new QName("http://test.x-road.fi/producer", "testServicePort");

String url = "http://localhost:8080/example-adapter-x.x.x-SNAPSHOT/Endpoint?wsdl";

// Create a service and add at least one port to it.
Service service = Service.create(SERVICE_NAME);
service.addPort(SERVICE_PORT, SOAPBinding.SOAP11HTTP_BINDING, url);

// Create a Dispatch instance from a service.
Dispatch<SOAPMessage> dispatch = service.createDispatch(SERVICE_PORT, SOAPMessage.class, Service.Mode.MESSAGE);

// Create SOAPMessage request. **/
MessageFactory mf = MessageFactory.newInstance(SOAPConstants.SOAP_1_1_PROTOCOL);

// Create a message.  This example works with the SOAPPart.
SOAPMessage request = mf.createMessage();
SOAPPart part = request.getSOAPPart();

// Obtain the SOAPEnvelope and header and body elements.
SOAPEnvelope env = part.getEnvelope();
SOAPHeader header = env.getHeader();
SOAPBody body = env.getBody();

env.addNamespaceDeclaration("id", "http://x-road.eu/xsd/identifiers");
env.addNamespaceDeclaration("xrd", "http://x-road.eu/xsd/xroad.xsd");

// Start by defining client soap header
SOAPElement clientHeader = header.addChildElement("client", "xrd");
clientHeader.addAttribute(env.createQName("objectType", "id"), "SUBSYSTEM");

SOAPElement xRoadInstance = clientHeader.addChildElement("xRoadInstance", "id");
xRoadInstance.addTextNode("NIIS-TEST");

SOAPElement memberClass = clientHeader.addChildElement("memberClass", "id");
memberClass.addTextNode("GOV");

SOAPElement memberCode = clientHeader.addChildElement("memberCode", "id");
memberCode.addTextNode("1234567-8");

SOAPElement subsystem = clientHeader.addChildElement("subsystemCode", "id");
subsystem.addTextNode("TestClient");


// Service soap header
SOAPElement serviceHeader = header.addChildElement("service", "xrd");
serviceHeader.addAttribute(env.createQName("objectType", "id"), "SERVICE");

xRoadInstance = serviceHeader.addChildElement("xRoadInstance", "id");
xRoadInstance.addTextNode("NIIS-TEST");

memberClass = serviceHeader.addChildElement("memberClass", "id");
memberClass.addTextNode("GOV");

memberCode = serviceHeader.addChildElement("memberCode", "id");
memberCode.addTextNode("1234567-8");

subsystem = serviceHeader.addChildElement("subsystemCode", "id");
subsystem.addTextNode("DemoService");

SOAPElement serviceCode = serviceHeader.addChildElement("serviceCode", "id");
serviceCode.addTextNode("helloService");

SOAPElement serviceVersion = serviceHeader.addChildElement("serviceVersion", "id");
serviceVersion.addTextNode("v1");

// Rest of the header elements
SOAPElement id = header.addChildElement("id", "xrd");
id.addTextNode("12345");

SOAPElement userId = header.addChildElement("userId", "xrd");
userId.addTextNode("jdoe");

SOAPElement protocolVersion = header.addChildElement("protocolVersion", "xrd");
protocolVersion.addTextNode("4.0");

// Construct the message payload.
SOAPElement operation = body.addChildElement("helloService", "ns1", "http://test.x-road.fi/producer");
SOAPElement value = operation.addChildElement("name");
value.addTextNode("Test User");
request.saveChanges();

// Invoke the service endpoint.
SOAPMessage response = dispatch.invoke(request);

// Get response: Body -> helloServiceResponse -> message -> <response>
logger.info("{}",response.getSOAPBody().getChildNodes().item(0).getChildNodes().item(0).getTextContent());
```

Sample code is available at https://github.com/nordic-institute/X-Road-code-samples/tree/master/full-samples/jaxws.

## JavaScript

### node-soap

[node-soap](https://github.com/vpulim/node-soap) is a Node module that lets you
connect to web services using SOAP.

This example demonstrates how to invoke ```getRandom``` service with no request parameters.
Sample client (```index.js```):

```
var soap = require('soap');
var url = 'http://localhost:8080/example-adapter-x.x.x-SNAPSHOT/Endpoint?wsdl';
var args = {};
soap.createClient(url, function(err, client) {

  client.addSoapHeader({
    "xrd:client": {
      "namespace": "xmlns:tns",
      xRoadInstance: 'NIIS-TEST',
      memberClass: 'GOV',
      memberCode: '1234567-8',
      subsystemCode: 'TestClient',
      attributes: {
        "id:objectType": 'SUBSYSTEM'
      }
    },
    "xrd:service": {
      xRoadInstance: 'NIIS-TEST',
      memberClass: 'GOV',
      memberCode: '9876543-1',
      subsystemCode: 'DemoService',
      serviceCode: 'getRandom',
      serviceVersion: 'v1',
      attributes: {
        "id:objectType": 'SERVICE'
      }
    },
    "xrd:id": 'ID11234',
    "xrd:userId": 'EE1234567890',
    "xrd:protocolVersion": '4.0'
    },
    null, null, null
  );
  client.getRandom(args, function(err, result) {
    console.log(result);
  });
});
```

How to create a simple client:

```
$ mkdir soap
$ cd soap
$ npm init -y
$ npm install soap
$ touch index.js
# Copy the code snippet below in index.js and update the adapter URL (x.x.x)
```

Make sure that the Example Adapter service is running and that the service
URL used in the code is correct. Then run the client:

```
$ node index.js
```

Example Adapter service returns a random number.

```
{ data: '8' }
```

Sample code is available at https://github.com/nordic-institute/X-Road-code-samples/tree/master/full-samples/node-soap.

## Credits

The material was originally created by Jarkko Moilanen (https://github.com/kyyberi) and Karri Niemelä (https://github.com/kakoni) in 2016. In January 2018 it was agreed that the [Nordic Institute for Interoperability Solutions](https://niis.org) (NIIS) takes over the responsibility for maintenance and further development.
