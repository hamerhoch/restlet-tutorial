# Restlet Framework Web API Reference Implementation

## Prerequisites

* Maven installed on your machine : [Maven documentation link](http://maven.apache.org/index.html)

> The implementation is located [here](https://github.com/restlet/restlet-tutorial/tree/master/modules/org.restlet.tutorial.webapi)

> This example uses [Restlet Framework 2.2.1](http://restlet.com/download/current#release=stable&edition=jse&distribution=zip) (Java SE edition)
and [H2 Database](www.h2database.com)

> Restlet framework user guide is available [here](http://restlet.com/learn/guide/2.3/).
> You can also clone this repository and go to ```/modules/org.restlet.tutorial.webapi/``` folder.

## Installation

### Install Maven project

To install maven :
* Go to the directory
* Execute ```mvn clean install```
* For eclipse users : run ```mvn eclipse:eclipse```

> For further instruction about running a Maven project : [Building a project with Maven](http://maven.apache.org/run-maven/index.html)

### Run this application

The main class is : ```WebApiHost.java```
By default the application is launched at ```http://localhost:9000```, but you can change it in ```WebApiHost.java```.
A logger is configured. Its configuration is made in ```logging.properties``` and declared in ```WebApiHost.java```.
Two files will be created :
 * ```webapitutorial-app-x-x.log```
 * ```webapitutorial-requests-x-x.log```

They are both situated in ```/tmp``` folder.
To simplify the launch of the application, authentication and authorization are done in-memory
(You should  overwrite that with you own authentication system). Here are the login/password available:

* admin/admin : to get admin role
* owner/owner : to get owner role
* user/user : to get user role

It uses HTTP Basic authentication.

> Learn more about authentication, authorization and security with Restlet Framework [here](http://restlet.com/learn/guide/2.3/core/security/).

> You can try this application easily with a REST client like [POSTMAN](http://www.getpostman.com/).

## Database access

To visualize the database, open the H2 console in you browser (`http://localhost:8082`) and connect to the database with the JDBC URL `jdbc:h2:mem:restletWebApi;IFEXISTS=TRUE`.

## Description

This Web API contains 2 main resources :

* Company : identified by an auto-generated id.
* Contact : identified by its email. A contact can be part of a company and get a reference to it.

This is a diagram of the API :

![Diagram](https://github.com/restlet/restlet-tutorial/blob/master/modules/org.restlet.tutorial.webapi/images/RFWebAPIReferenceImplementation.png)

> A Web API definition can be generated with [APISpark extension (RF 2.3)](http://restlet.com/learn/guide/2.3/extensions/apispark).

## Implementation choices

 * The persistence layer has been done to be as simple as possible. The aim of this example is to show how to build a reference Web API with Restlet Framework.
 * The API was designed to use meaningful URLs. For example, for contacts the email is used as an identifier instead of a technical id (which does not mean anything for a user). On the contrary, for companies an autogenerated id has been chosen.
 * There is not POST method for contacts as a contact is defined by his/her email. To add a contact, you must do a PUT on ```/contacts/{email}```

## Next steps

Here are some instructions to go further with this project

 * Persistence layer
 	* For each operation a new connection is created.
 	It would be useful to use a connection pool like [DBCP](http://commons.apache.org/proper/commons-dbcp/).
 	* It is possible to totally dissociate the persistence layer from the Server Resources.
 	It would allow the use of several persitence layers. It would also be possible to use dependency injection.
 * Authentication/Authorization
 	* It is strongly recommended to store users/roles in a database instead of in-memory.
 	You can create a custom [SecretVerifier](http://restlet.org/learn/javadocs/2.2/jse/api/org/restlet/security/SecretVerifier.html)
 	 and [Enroler](http://restlet.org/learn/javadocs/2.2/jse/api/org/restlet/security/Enroler.html).
 	* It is possible to handle autorizations in a filter instead of at the beginning of each class.
 	In fact, in this example, there is a repeating schema : Role user needed to access to GET methods,
 	Role owner needed to access the others.

## Usage

> These examples are made using the JSON format but you can use XML or YAML if you want.

### Ping resource

A resource ```/ping``` has been created which does not need authentication.

```GET 		http://localhost:9000/v1/ping```

![Ping Example](https://github.com/restlet/restlet-tutorial/blob/master/modules/org.restlet.tutorial.webapi/images/ping.png)

> For the following examples, Basic Authentication will be needed

### Create a company

 ``` POST	http://localhost:9000/v1/companies/```

 ```json
{
  	"name" : "Restlet",
  	"duns" : "123456789",
  	"address" : "2 API road",
  	"zipCode" : "44300",
  	"creationDate" : "2014-01-01",
  	"website" : "http://restlet.com",
  	"city" : "Nantes",
  	"phoneNumber" : "+1 555 555 555"
}
```

![POST Companies](https://github.com/restlet/restlet-tutorial/blob/master/modules/org.restlet.tutorial.webapi/images/POSTCompanies.png)

> The returned status is : ```201 Created```.

The location of the created company is written is the "location" header.

 ![POST Companies headers](https://github.com/restlet/restlet-tutorial/blob/master/modules/org.restlet.tutorial.webapi/images/POSTCompaniesHeaders.png)

### Retrieve all created companies

 ```GET		http://localhost:9000/v1/companies/```

> The trailing slash is optional : both ```http://localhost:9000/v1/companies/``` and ```http://localhost:9000/v1/companies``` will work.

 It should retrieve :

 ```json
{
	"list" :
		[
			{
				"duns":"123456789",
				"address":"2 API road",
				"zipCode":"44300",
				"creationDate":"2014-01-01",
				"website":"http://restlet.com",
				"phoneNumber":"+1 555 555 555",
				"city":"Nantes",
				"name":"Restlet",
				"self":"/companies/1"
			}
		]
}
```

![GET Companies](https://github.com/restlet/restlet-tutorial/blob/master/modules/org.restlet.tutorial.webapi/images/GETCompanies.png)

> ```self``` element refers to the location of the object : ```http://localhost:9000/v1/companies/1```.
> Try ```GET	http://localhost:9000/v1/companies/1``` !

### Create a contact related to the created company

 ```PUT	http://localhost:9000/v1/contacts/{email}```

  with ```{email}``` representing the email of the contact you want to insert.

 For this example, it is :
 ``` PUT	http://localhost:9000/v1/contacts/gblondeau@restlet.com```

 ```json
 {
 	"name" : "Blondeau",
 	"firstName" : "Guillaume",
 	"age" : 22,
 	"company" : "1"
 }
 ```

 It should retrieve :

 ```json
 {
	"email":"gblondeau@restlet.com",
	"age":22,
	"name":"Blondeau",
	"firstName":"Guillaume",
	"company":"/companies/1"
}
 ```

 ![PUT Contacts](https://github.com/restlet/restlet-tutorial/blob/master/modules/org.restlet.tutorial.webapi/images/PUTContacts.png)

> The returned status is : ```201 Created```.

> The field company is a reference to the location of the company, with 1 the id of the company.
 	ie : /companies/1 refers to http://localhost:9000/v1/companies/1

### Retrieve the list of created contacts

```GET		http://localhost:9000/v1/contacts```

> The trailing slash is optional : both ```http://localhost:9000/v1/contacts/``` and ```http://localhost:9000/v1/contacts``` will work.

It should retrieve :

```json
{
	"list" :
		[
			{
				"email":"gblondeau@restlet.com",
				"age":22,
				"name":"Blondeau",
				"firstName":"Guillaume",
				"company":"/companies/1",
				"self":"/contacts/gblondeau@restlet.com"
			}
		]
}
```

  ![GET Contacts](https://github.com/restlet/restlet-tutorial/blob/master/modules/org.restlet.tutorial.webapi/images/GETContacts.png)

> The field company is a reference to the location of the company

To display the company information, add the following query parameter : ```?strategy=load```

```GET		http://localhost:9000/v1/contacts/?strategy=load```

It should retrieve :

```json
{
	"list" :
		[
			{
				"email":"gblondeau@restlet.com",
				"age":22,
				"name":"Blondeau",
				"firstName":"Guillaume",
				"company":
					{
						"duns":"123456789",
						"address":"2 API road",
						"zipCode":"44300",
						"creationDate":"2014-01-01",
						"website":"http://restlet.com",
						"phoneNumber":"+1 555 555 555",
						"city":"Nantes",
						"name":"Restlet"
					},
				"self":"/contacts/gblondeau@restlet.com"
			}
		]
}
```

  ![GET Contacts](https://github.com/restlet/restlet-tutorial/blob/master/modules/org.restlet.tutorial.webapi/images/GETContactsLoad.png)

### Modify an existing contact

```PUT		http://localhost:9000/v1/contacts/gblondeau@restlet.com```

```json
{
    "age" : 23,
    "name": "Blondeau",
    "firstName": "Guillaume",
    "company": "1"
}
```

It should retrieve :

```json
{
    "email": "gblondeau@restlet.com",
    "age": 23,
    "name": "Blondeau",
    "firstName": "Guillaume",
    "company": "/companies/1"
}
```

  ![PUT Contacts](https://github.com/restlet/restlet-tutorial/blob/master/modules/org.restlet.tutorial.webapi/images/PUTContacts2.png)

> This time a ```200 OK ``` is returned because it is not a creation but an update.

