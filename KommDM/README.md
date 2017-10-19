# Motorjourney and CISL

This repository contains 2 projects.
The first project is a angular application which shows a demo journey (Motorjourney)
The second project is the CISL Adapter.
The Angular application shows the connectivity between a angular application and CISL, both running on CloudFoundry.

DemoApp:
https://motor-journey.apps.adp.allianz/

CISL:
https://cisl-adapter.apps.adp.allianz/

## Table of contents

* [Table of contents](#table-of-contents)
* [Introduction](#Introduction)
	* [Purpose](#purpose)
	* [Solution](#solution)
	* [Visionary Scenario](#visionary-scenario)
* [System Design](#Analysis)
	* [Component Diagram](#component-diagram)
	* [Access Control and Security](#access-control-and-security)
* [Demo Scenario](#demo-scenario)
* [Administrator Manual](#administrator-manual)
	* [Infrastructure Setup](#infrastructure-setup)
	* [Deployment and Configuration](#deployment-and-configuration)
* [Contact Details](#contact-details)
	

## Introduction

### Purpose

This repository shows how to connect a angular web application to CISL. The main goal of this project is to provide a configuration for applications that will retrieve data from CISL.

The angular web application implements a basic journey "Motor Quote and Buy".

CISL and the DemoApp are hosted on CloudFoundry. 

### Visionary Scenario

With the provided configurations any application will be able to retrieve data from CISL.

## System Design

### Component Diagram

// TODO

### Access and Control Security

#### Authentication

The CISL calls are restricted by a bearer token. Bearer Token:
A security token with the property that any party in possession of the token (a "bearer") can use the token in any way that any other party in possession of it can. Using a bearer token does not require a bearer to prove possession of cryptographic key material (proof-of-possession).
See the UserService for more details.

To get the Bearer Token you can open the SwaggerUI and copy the Bearer token. The Token can be configured in the application.yml, such as the duration of the validity of the token. The token is preconfigured to be valid for 30 minutes. Example: tokenValidityInSeconds: 1800

#### Infrastructre

CloudFoundry uses for both instances (DemoApp and CISL) a secure communication protocol: HTTPS (also called HTTP over Transport Layer Security [TLS], HTTP over SSL, and HTTP Secure).
HTTPS URLs begin with "https://" and use port 443 by default, whereas HTTP URLs begin with "http://" and use port 80 by default.

If you run CISL locally the port to access the API is 8090 (http://localhost:8090/) That why the port has to be mapped internally to 443. 
The port can be changed:
* Directly in the code: SwaggerSpec.java --> change Port to 443
* In the groovy file as an Enviroment varible of CF: -Dserver.port=443

#### Accessibility 

The API is wrapped with a basic "login"-authentification. To turn this authentification off the application.yml needs to be changed:
* 1. There is application.yml inside the adapter: /src/main/resources/config;
* 2. Some other .yml files are available and used in case a different spring profiles is chosen.This is set in the groovy file as an Enviroment varible of CF: -Dspring.profiles.active=myProfile. 
* 3. You can provide an external .yml file in command line when you run the adapter (not sure if you can pass it in CF); This is set in the groovy file as an Enviroment varible of CF: -Dspring.config.location=pathToSomeYmlFile.yml
For more details see the groovy file

CORS-Policy

Cross-origin resource sharing (CORS) is a mechanism that allows restricted resources (e.g. fonts) on a web page to be requested from another domain outside the domain from which the first resource was served. A web page may freely embed cross-origin images, stylesheets, scripts, iframes, and videos. Certain "cross-domain" requests, notably Ajax requests, however, are forbidden by default by the same-origin security policy. https://developer.mozilla.org/en-US/docs/Web/HTTP/Access_control_CORS


To bypass the CORS-policy the DemoApp the headers of the call need to be adapted:

```
	// Create the XHR object.
    $scope.createCORSRequest = function(method, url) {
      var xhr = new XMLHttpRequest();
      if ("withCredentials" in xhr) {
        // XHR for Chrome/Firefox/Opera/Safari.
        xhr.open(method, url, true);
      } else if (typeof XDomainRequest != "undefined") {
        // XDomainRequest for IE.
        xhr = new XDomainRequest();
        xhr.open(method, url);
      } else {
	console.log('CORS not supported.');
        xhr = null;
      }
      return xhr;
    }

    // Make the actual CORS request.
    $scope.makeCorsRequest = function(url, token, func) {
      // This is a sample server that supports CORS.

      var xhr = this.createCORSRequest('GET', url);
      if (!xhr) {
        alert('CORS not supported');
        return;
      }

      // Response handlers.
      xhr.onload = function() {
	  console.log("Response: ", xhr.response);
      };

      xhr.onerror = function() {
        console.log('Woops, there was an error making the request.');
      };
      xhr.setRequestHeader(
          'Authorization', token);
      xhr.send();
    }
```

Making this CORS-Request there is no more need to take care of SSL certificate validation.

## Demo Scenario

//TODO: Screenshots.

## Administrator Manual

### Infrastructure Setup

#### Jenkins

These are the configurations of the Jenkins Pipeline:

![image](Motorjourney-Angular/config/config1.JPG)

![image](Motorjourney-Angular/config/config2.JPG)

Before building and running the application please make sure that these folders are in the .cfingore:

```
node_modules/
groovy/
```
Specially "groovy" otherwise the app will crash

#### CloudFoundry

As mentioned before the DemoApp and CISL are deployed to CF. The DemoApp is an instance called "motor-journey" and CISL is another instance called "cisl-adapter". 
Both run in the same space "CISL Demo". Since both apps are build and deployed from Jenkins there is no need to use the CF Client.

Before building and running the application please make sure that these folders are in the .cfingore:

```
node_modules/
groovy/
```
Specially "groovy" otherwise the app will crash

#### Requirements

* Access to CISL
* Installation of NodeJS (to test DemoApp locally)

#### Installation

##### Getting Started

To get you started you can simply clone the repo

```
git clone https://github.developer.allianz.io/AZDBusinessArchitecture/CISL-Adapter.git
```

##### Install Dependencies

Dependencies are installed from Nexus. https://cisl.allianz.at/nexus/repository/cisl-releases/

To be able to install these the NO_PROXY Variable has to be set in the Enviroment of CloudFoundry:
```
env.NO_PROXY = 'localhost,127.0.0.1,10.17.175.,172.30.,172.31.,.ghe.adp.allianz,.cluster.local,.cisl.allianz.at,cisl.allianz.at,.allianz.cisl'
```

##### Run the Application

Both Instances are hosted in CF. They are build from Jenkins and deployed with the groovy-script to CF. They are started automatically.
To access both instances please open the links above.

###### DemoApp:

To test the DemoApp locally just run the following cmd to 
We have preconfigured the project with a simple development web server.  The simplest way to start
this server is:

```
node index.js
```

Now browse to the app at `http://localhost:3000/index.html`.

This scenario will describe the customer journey for a basic motor quote and buy.

Offerings, Contracts, Policies and Questions are needed for this scenario.

To keep the scenario simple some of the step of motoquote have been ignored. For further details see: cisl.allianz.at -> Customer Journey: Basic Motor Quote and Buy

Calculation is done on the frontend.

###### CISL-Adapter:

If you want to run the CISL-Adapter locally follow the next steps:

1. Clone Cisl-Repo:

``
git clone https://cisl.allianz.at/cisl/cisl.git
``

2. Maven clean install:

``
mvn clean install
``

3. Change directory and run:

```
cd adapter/com.allianz.cisl.adapter.boot/

mvn spring-boot:run
```

The CISL API is available at http://localhost:8090/api

##### Offerings 

The user first gets an overview of the available offers of class "Motor" since its a scenario for motor quotations.

The policies can be retrieved by the call:
``
http://localhost:8090/api/offerings
``
A description for the different types of classes should be given.

##### Contracts

CISL supports a broad range of quote and buy related functionalities. As with all core insurance processes supported by CISL a root resource /contracts and /contracts/{contractId} is available to enter the process via the POST operation. Within the “contract” object an enumeration “ContractProcessStadium” is available to further distinguish between a quote “QUOTATION”, the application phase “APPLICATION” and the contract handling phase “CONTRACT”. Depending on the capabilities of the individual backends specific services might only be available in a specific state of the process. Furthermore there might also be different validations triggered depending on the state of the contract. For Backends which do not distinguish between the states mentioned above the enumeration “UNSPECIFIED” was introduced.

##### Calculation

The calculation is done in the frontend but should be calculated throughout the following call:
/questions and /questions/{questionId} are the main services to work with process related questions towards a customer. These services are used within all relevant business processes like claims, contracts, and business acceptance. For this services there is only a put available since question and answer are always bundled together to ensure data integrity. Therefore it is assumed that the question and possible answer (type) is already available on the backend and the customer will only provide the content (the actual answer). As with other functionality this services can be used for multiple question types like, Boolean, date, free text, datalist questions. The questions themselves can contain sub questions and can be bundled into catalogues when multiple categories are needed. With these functionalities it is possible to transport dynamic decision trees to a front end. Within every question there are as described above one or more detail answer objects available. These objects contain the answer and information about the decision tree if one is available.

##### API Calls

The API calls have been implemented in the Main service.
For more details: src/services/main/main.service.js

### Deployment and Configuration

#### Deployment

Both instances are deployed from Jenkins via a groovy Script with the mentioned configurations. For detailed information take a look at the groovy-files.

#### Configuration

Before building CISL the predefined Data can be preconfigured using JSON-Files. Depending on the json-file the path to store it is different. The main path to the configurable jsons is:
```
src/main/resources/init-data/
```
In this case the Questions for the policy are configured:

##### Questions

Questions should be implemented by the provider. For this example a single question has been implemented hardcoded.
/questions and /questions/{questionId} are the main services to work with process related questions towards a customer. These services are used within all relevant business processes like claims, contracts, and business acceptance. For this services there is only a put available since question and answer are always bundled together to ensure data integrity. Therefore it is assumed that the question and possible answer (type) is already available on the backend and the customer will only provide the content (the actual answer). As with other functionality this services can be used for multiple question types like, Boolean, date, free text, datalist questions. The questions themselves can contain sub questions and can be bundled into catalogues when multiple categories are needed. With these functionalities it is possible to transport dynamic decision trees to a front end. Within every question there are as described above one or more detail answer objects available. These objects contain the answer and information about the decision tree if one is available.

```
[
  {
    "self": "V2EE3F52FD9EC0DE8A4651BDF7C201872425660B388D7041E2242E26CBB5E4874CDFE04163B0A37ED11B07D56D916CD904",
    "sign": "DSCHUTZ",
    "text": "Verwendung der Personen- und Vertragsdaten",
    "answer": {
      "self": "V2EE3F52FD9EC0DE8A4651BDF7C201872425660B388D7041E2242E26CBB5E4874CDFE04163B0A37ED11B07D56D916CD904",
      "eClass": "http://allianz.com/com.allianz.cisl.core.questions#//BooleanAnswer"
    },
    "eClass": "http://allianz.com/com.allianz.cisl.core.questions#//DecisionQuestion",
    "detailText": "Alle Antragsteller und zu versichernden Personen stimmen zu, dass ihre Personenidentifikationsdaten (wie z.B. Name, Adresse) und Vertragsdaten (Produkt, Leistungsumfang, Laufzeit) - keinesfalls jedoch sensible Daten - von der Allianz Elementar Versicherungs-AG, Allianz Elementar Lebensversicherungs-AG, der Top Versicherungsservice GmbH, der AGA International S.A. oder der Allianz Investmentbank AG einerseits zur Beratung und Betreuung (z.B. Vertragsanpassungen) sowie zwecks Zusendung von Marketingaktionen (z.B. Bonusgarantien, Tankgutscheine) und Produktvorschlägen (entweder per Post, Fax, E-Mail oder telefonisch) verwendet werden dürfen. Keinesfalls werden diese Daten an andere Unternehmen weitergegeben oder weiterverkauft. Diese Zustimmung kann jederzeit ohne Angabe von Gründen widerrufen werden.",
    "questionType": "X",
    "answerObligationType": "W",
    "questionSortingNumber": 100
  },
  {
    "self": "V2B4C821765BDC3ACD5E7157437E0B91AF25660B388D7041E2242E26CBB5E4874CDFE04163B0A37ED11B07D56D916CD904",
    "sign": "MAS1",
    "text": "DieersteMasterfrage",
    "answer": {
      "self": "V2B4C821765BDC3ACD5E7157437E0B91AF25660B388D7041E2242E26CBB5E4874CDFE04163B0A37ED11B07D56D916CD904",
      "value": false,
      "eClass": "http://allianz.com/com.allianz.cisl.core.questions#//BooleanAnswer"
    },
    "eClass": "http://allianz.com/com.allianz.cisl.core.questions#//DecisionQuestion",
    "detailText": "DieersteMasterfrage",
    "questionType": "X",
    "answerObligationType": "J",
    "questionSortingNumber": 3
  },
  {
    "self": "V2ACB23E8254F5019F5A02D62C12C5C82925660B388D7041E2242E26CBB5E4874CDFE04163B0A37ED11B07D56D916CD904",
    "sign": "ABGEL",
    "text": "Wurden Versicherungen abgelehnt?",
    "answer": {
      "self": "V2ACB23E8254F5019F5A02D62C12C5C82925660B388D7041E2242E26CBB5E4874CDFE04163B0A37ED11B07D56D916CD904",
      "value": false,
      "eClass": "http://allianz.com/com.allianz.cisl.core.questions#//BooleanAnswer"
    },
    "eClass": "http://allianz.com/com.allianz.cisl.core.questions#//DecisionQuestion",
    "detailText": "Wurden Ihnen zu diesem KFZ bereits einmal Versicherungen abgelehnt?",
    "questionType": "X",
    "answerObligationType": "J",
    "questionSortingNumber": 1
  },
  {
    "self": "V2E2FACD685AE0D1E3F3806E89586A074125660B388D7041E2242E26CBB5E4874CDFE04163B0A37ED11B07D56D916CD904",
    "sign": "SCHRIFTFORM",
    "text": "Vereinbarung der Schriftform",
    "answer": {
      "self": "V2E2FACD685AE0D1E3F3806E89586A074125660B388D7041E2242E26CBB5E4874CDFE04163B0A37ED11B07D56D916CD904",
      "value": true,
      "eClass": "http://allianz.com/com.allianz.cisl.core.questions#//BooleanAnswer"
    },
    "eClass": "http://allianz.com/com.allianz.cisl.core.questions#//DecisionQuestion",
    "detailText": "Ich habe die nachfolgende in diesem Antrag enthaltene Vereinbarung zur Form von Erklärungen und anderen Informationen gelesen und stimme ihr zu.",
    "questionType": "X",
    "answerObligationType": "W",
    "questionSortingNumber": 500
  },
  {
    "self": "V2851F96F92F25341C4FA7698C5B96D43925660B388D7041E2242E26CBB5E4874CDFE04163B0A37ED11B07D56D916CD904",
    "sign": "KUEND",
    "text": "Zu kündigende Verträge vorhanden ?",
    "answer": {
      "self": "V2851F96F92F25341C4FA7698C5B96D43925660B388D7041E2242E26CBB5E4874CDFE04163B0A37ED11B07D56D916CD904",
      "value": false,
      "eClass": "http://allianz.com/com.allianz.cisl.core.questions#//BooleanAnswer"
    },
    "eClass": "http://allianz.com/com.allianz.cisl.core.questions#//DecisionQuestion",
    "detailText": "Bestehen zu den beantragten Risiken noch zu kündigende Verträge?",
    "questionType": "X",
    "answerObligationType": "J",
    "questionSortingNumber": 2
  }
]
```

The questions above can be changed regarding the needs for the journey. The "self"-value within "answer" are the answer-objects that are also preconfigured. Therefore it is assumed that the question and possible answer (type) is already available on the backend and the customer will only provide the content (the actual answer).. As with other functionality this services can be used for multiple question types like, Boolean, date, free text, datalist questions. The questions themselves can contain sub questions and can be bundled into catalogues when multiple categories are needed. With these functionalities it is possible to transport dynamic decision trees to a front end. Within every question there are as described above one or more detail answer objects available. These objects contain the answer and information about the decision tree if one is available.

## Contact Details

For more information please contact:

[Martin Leichter](#https://github.developer.allianz.io/martin-leichter)
[Nicolas Corpancho Villasana](#https://github.developer.allianz.io/nicolas-corpancho-villasana)

## Feedback

The Dev-Team of CISL provided the information about configuring data (questions.json). The Team included this information in the CISL Documentation after requesting the information on how to configure data on the API.





