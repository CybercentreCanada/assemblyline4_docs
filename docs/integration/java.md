# Java Client

The Assemblyline Java client library provides methods to submit requests to Assemblyline.

:octicons-mark-github-16: [Get the Java client](https://github.com/CybercentreCanada/assemblyline-java-client)

## Using the client

To instantiate the client bean set the application properties associated with the desired authentication method. The client can be accessed by auto-wiring the bean into the class using it.

There are two authentication methods: username/apikey or username/password.

### API Key Authentication

To instantiate an API key authenticated Assemblyline client, define the following properties:

    assemblyline-java-client:
        url: <assemblyline-instance-url>
        api-auth:
            apikey: <api-key>
            username: <username>

### Password Authentication

To instantiate a password authenticated Assemblyline client, define the following properties:

    assemblyline-java-client:
        url: <assemblyline-instance-url>
        password-auth:
            password: <password>
            username: <username>

### Proxy

To go through a proxy, add the following properties:

    assemblyline-java-client:
        proxy:
            host: <host>
            port: <port>
