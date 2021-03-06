---

title: 'Security'
weight: 210

menu:
  main:
    identifier: "user-guide-security"
    parent: "user-guide"

---

This page looks at Camunda BPM from a security perspective. It provides an overview over how to secure a Camunda installation.
In order to secure a Camunda installation, Camunda itself must be configured correctly and it must be integrated correctly into its _environment_.

# Reporting Security Issues and Vulnerabilities

Making Camunda as secure as possible is an ongoing effort. If you discover a potential security vulnerability please report it through the [Jira Issue Tracker](http://app.camunda.com/jira).

# Deployment Options and Components

There are different ways of using Camunda BPM and different components are provided: the process engine itself, the REST Api, the web applications. Depending on how Camunda is deployed and which components are used, different security considerations apply. The following list gives a general overview over deployment options and components and points out the main differences from a security point of view. The remainder of this chapter then goes into different configuration options.

* Embedded Java library inside an application: in this case, the Camunda engine is embedded inside a custom Java Application. Usually the application takes care of securing access to Camunda's APIs and the APIs are not directly exposed to an end user. In this case, the application typically takes care of ensuring authentication and preventing access by unauthorized users.
* Shared Process Engine: in this scenario, the Process Engine is deployed as a container service into an application server such that it can be used by the applications deployed into the same container / server. This case is similar to the embedded Java library case.
* REST Api: the REST API provides access to Camunda's core APIs through HTTP. In this case users can directly access Camunda's APIs. Usually, it is necessary to configure authentication, authorization and also secure the connection to the REST API using SSL (HTTPS).
* Web Applications (Cockpit, Tasklist, ...): similar considerations to the REST API apply.

# Security Configuration inside Camunda

Camunda provides a number of configuration options which are relevant from a security perspective. Most prominently: authentication, authorization and the control of custom code (scripts) which can be executed on the server.

## Authentication

Authentication controls _who_ can access Camunda's APIs and Applications.

### Do I need Authentication?

Authentication is only needed in the following cases:

* Camunda's REST API is used
* Camunda's Web Applications are used

In these cases, direct access to Camunda's core APIs is provided over HTTP and authentication must be enabled.

By contrast, authentication is generally not done by Camunda when embedded as a library into an application. In this case the application takes care of authentication itself.

### Enabling Authentication for the REST API

For ease of use by developers, the REST API's authentication is disabled by default. When deploying the REST Api in production, it is therefore required to enable authentication. Check the corresponding section in the [REST Api documentation]({{< relref "reference/rest/overview/authentication.md" >}}).

### Authentication in the Web Applications

For the web applications, authentication is enabled by default and it is not possible to disable it.

### Internal (database backed) User Management

To perform authentication, Camunda can use two sources: a database or LDAP.

When using the the database, usernames and passwords are stored inside the `ACT_ID_USER` table (see [documentation on database schema]({{< relref "user-guide/process-engine/database.md#identity" >}})). To protect the passwords stored in the database, Camunda uses two concepts:

* **hashing**: instead of storing the password in plain text, a hash is stored. When authenticating, the same hash is generated from the user's input and compared against the hash in the database. If both hashes are equal the authentication attempt is successful. Camunda allows users to configure and customize the hash function used. Please refer the [documentation section on password hashing]({{< relref "user-guide/process-engine/password-hashing.md" >}}) for details.
* **salted hashes** to protect the database against rainbow table attacks, Camunda uses salted hashes. Similar to hashing itself, this function can be configured and extended to a user's needs. Please refer the [documentation section on password hashing]({{< relref "user-guide/process-engine/password-hashing.md" >}}) for details.

### LDAP

As an alternative to the database, Camunda can use LDAP for verifying user credentials on authentication. Camunda has read-only access to LDAP.

## Authorization

Authorization controls what data a user can access and change in Camunda once authenticated. Authentication is a pre-requisite to authorization.

### Do I need to enable Authorizations?

Similar considerations as for authentication apply. For an in-depth discussion, see the documentation section on [authorizations]({{< relref "user-guide/process-engine/authorization-service.md#when-is-authorization-required" >}})

### Restricting Data Access with Authorizations

Authorizations can be used to restrict a user from accessing a data object (such as a process or a task) and can be used to restrict how the user can interact with such data objects (read-only vs. modifications). Authorizations in Camunda are very powerful and it is recommended to read the corresponding [documentation entry on authorizations]({{< relref "user-guide/process-engine/authorization-service.md" >}}).

## Script Execution

Camunda allows users to deploy scrips to be used by BPMN processes or DMN decision tables. This is a very flexible and powerful feature and facilitates changing business logic fast since generally scripts can be deployed at runtime without restarting the server.
However, script languages such as Groovy or Javascript are executed directly inside the same JVM which also hosts Camunda itself. It is not easily feasible to sandbox the execution of such scripts effectively which is why deployment of such scripts should only be allowed to trusted users. To achieve this:

* Control who can deploy scripts through the appropriate CREATE [authorizations]({{< relref "user-guide/process-engine/authorization-service.md#resources" >}}) on the DEPLOYMENT resource.
* Consider disabling execution of scripts all together if the feature is not needed. See also: [Custom Code & Security
]({{< relref "user-guide/process-engine/securing-custom-code.md" >}})

## Expressions in Queries

Consider disabling execution of expressions in queries. See also: [Custom Code & Security
]({{< relref "user-guide/process-engine/securing-custom-code.md" >}})

# Security Configuration in the external Environment

Camunda integrates into an environment, most prominently the database and, when using the web applications or the REST API, also a webserver. In order to secure your Camunda deployment as a whole, the integration is relevant.

## Database

Camunda stores its data into a relational database. In order to protect access to this data, it must be configured correctly.
The documentation section on [supported environments]({{< relref "introduction/supported-environments.md" >}}) provides a list of supported databases.

### Data encryption

To prevent unauthorized access to the data stored in the Camunda database you must follow best practices around operating the database, including data encryption. Please make sure to consult the manual provided by your database vendor.

### Securing the database connection

To access the database, Camunda needs to establish a connection. Usually the connection is configured either directly through the JDBC configuration options or through a _datasource_ configured inside the application server. Most database drivers support encrypted connections and transport layer security when connecting to the database. When operating Camunda and the database in an untrusted network it is recommended to enable these features. Please consider the manuals of your database, database driver and your application server to do so.

### Securing Database Credentials

To establish the connection to the database, the database credentials need to be provided. As opposed to providing the credentials as plain text in a configuration file, some application servers support storing the credentials securely in an encrypted form. In that case, consult the manual of your application server to learn how to use these features.

## Web Server (applicable when using REST Api or Web Applications)

When deploying the REST API or the Camunda Web Applications, Camunda is integrated with a third party web server. The documentation section on [supported environments]({{< relref "introduction/supported-environments.md" >}}) provides a list of supported web servers. / application servers

### Enabling SSL / HTTPS

It is strongly recommended to configure SSL / HTTPS when deploying the Camunda REST APIs or Web Applications. This can be achieved by configuring HTTPS either on the web server itself or through a reverse proxy. Please consult the manual of your web server or reverse proxy for details.
