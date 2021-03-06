# README #

# sri-client #

* this is the project with all kind of utilities for clients which are using  A [sri4node][sri4node-project] API.
* version 1.1.2

## sri-clients ##
The project contains two client modules with all kind of functions to do API requests according to [SRI (Standard ROA Interface)][sri-documentation]:

* an Angular 1 module
* a node module

They both have the same generic interface

### generic interface ###

* **get(href, parameters, options):** http get of the href with the given parameters. Returns a promise with the exact result from the api.
* **getList(href, parameters, options):** http get of the href with the given parameters where href is suposed to be a list resource.
Returns a promise with the array of the expanded results that match the query (so not an object with an href and $$expanded, but the object that is $$expanded.
The list of results is limited to only one API call so the lenght will be maximum the limit. The result also has a method count() which returns the count from the $$meta section.
* **getAll(href, parameters, options):** http get of the href with the given parameters where href is suposed to be a list resource.
Returns a promise with the array of the expanded results that match the query (so not an object with an href and $$expanded, but the object that is $$expanded.
The list of results is all the results that match the query, because the next links are requested as well and concatenated to the result.
The result also has a method count() which returns the count from the $$meta section.
* **put(href, payload, options):** http put to href with the given payload.
* **updateResouce(resource, options):** http to resource.$$meta.permalink with resouce as payload. Compact function to do an update of an existing resource.
* **post(href, payload, options):** http post to href with the given payload.
* **delete(href, options):** http delete to href.
* **getAllHrefs(hrefs, batchHref, parameters, options):** returns an array of all objects for hrefs, a given array with permalinks.
All these parameters need to be of the same resource type! You can provide expansion (or other) parameters with parameters.
It will get all these permalinks in the most efficient way if an href to the corresponding batch url is provided.
If the batch url is null it will get them in individual request in groups of 100  (can be overwritten with options.groupBy) permalinks in order to not make the request url too long.
* **getAllReferencesTo(baseHref, params, referencingParameterName, hrefsArray, options):** Same as getAll but you can add a referencingParameterName and an array of hrefs.
It will add referencingParameterName as a parameter and add the hrefsArray as a comma separated string,
but it will only request them in groups of 100 (can be overwritten with options.groupBy) to make sure the request url does not get too long.
If the name of the resource is too long you might have to use options.groupBy to decrease the number of hrefs it groups in one request

All these methods return a promise. If the response status >= 400, the result will return an error object with:

* **status:** the http status of the response, null if there was no response.
* **body:** the body of the response if it was present.
* **getResponseHeader(headerName):** method that returns the value of the given headerName of the response.

If the result of put, updateResouce or post was < 300 the promise returns an object with:

* **getResponseHeader(headerName):** method that returns the value of the given headerName of the response.

All methods have an **options** object that you can pass on as a parameter. You can specify the following properties:

* **common**
  * **baseUrl:** sends the http request to this baseUrl instead of the default baseUrl that is set in the initialisation of the configuration.
  * **headers:** An object with headers that are added on the http request. f.e.: {'foo': 'bar'} adds a header foo with value bar.
  * **expand:** array with property paths that you want to expand client side. You can expand as deep as you want and don't need to add $$expanded to the path. See the examples.
  You can also replace a property path string with an object to pass on more advanced options. The object contains the following proerties:
    * property: [required] the property path.
    * required: default is true. If true an error will be thrown if a property in the property path can not be found. This is to help the user detect typo's.
    If required is false, no error will be thrown and expansion will be ignored for objects that do not have a property in the property path. If the property path contains properties that are not required you need to set this proeprty to false.
    * include: you can include resources within the expanded resources. Works the same as the include property on the root of the options.
  Example: api.getAll('/responsibilities/relations', {to: #{an href to a responsibility}}, {expand: [from.organisationalunit]}) expands in two steps first all froms in the resultset, and in a second step all organisationalunits of those froms.
  * **include:** array of objects with configuration to include a property of the/all resrouce(s) (in the resultset). The configuration can contain the following properties:
    * alias: [required] An {{alias}} property will be added to the resource or every resource in the resultset if it is a list. It is recommended to add a $$ prefix in front of them so the client will not send this back to the server on PUT.
    * href: [required] The href on which the included resources can be found
    * reference: an object with the following properties:
      * property:  [required] The property name that references $$meta.permalink of the resource for which you are including resources
      * parameterName: This is the parameter name on the href to reference the $$meta.permalink of the resource you are including resources.
      If not added the property name will be taken as parameter name because in many cases this is the same. There is also a short notation where the value of reference is just the property name as a string instead of an object.
    * filters: filter on the resources you are including (for example include only external identifiers of type INSTITUTION_NUMBER when you are including external identifiers in an organisational unit)
    * expanded: true is default so you don't need to add this property if true. If false all results will be unexpanded and you will get only the hrefs
    * singleton: false is default so you don't need to add this property if false. The value of '$${{alias}}' will be an object (the first object in the resultset) instead of an array. It will be null if there are no results.
    * expand: client side expansion of properties within the included resources. Works the same as the expand property on the root of the options.
    * include: include resources within the included resources. This is just recursive and works the same as the include property on the root of the options.
  * **limit:** return a limited number of results for getList, the limit can also be higher than the maximum limit that the server allows. =TODO
  * **raw:** boolean that can be used for getList and getAll. If set to true the array of results will contain the raw result of the body.results, so the items will be an object with an href and a $$expanded object.
  * **asMap:** boolean that can be used for getAllHrefs. If set to true an object with be returned which contains all the hrefs as keys and the object to which it refers as value.
* **ng-sri-client specific**
  * **cancelPromise:** A promise that will cancel the request, if it gets resolved.
  * **pending:** boolean that will be set to true by every method when the request starts and set to false once the result is fetched.
* **node-sri-client specific**
  * **timeout:** The number of miliseconds to wait before the request times out. (default timeout is 10 seconds for a GET, 30 seconds for a sendPayload and 120 seconds for a batch)
  * **maxAttempts:** The number of times the request will be attempted. The default is 3 times (so 2 retries).
  * **retryStrategy:** You can pass on your own strategy on when to retry a request. The default is to retry on network errors (ECONNRESET, ENOTFOUND, ESOCKETTIMEDOUT, ETIMEDOUT, ECONNREFUSED, EHOSTUNREACH, EPIPE, EAI_AGAIN) or on http errors with status 5xx. See the documentation of [requestretry][npm-requestretry-strategy] to see how to define your own delay strategy.
  * **retryDelay:** The number of miliseconds of delay untill another attempt is made for this same request. The default is 5000 miliseconds.
  * **delayStrategy:** You can pass on your own strategy on how to delay a request. The default is to have a fixed time (options.retryDelay) in between the requests. See the documentation of [requestretry][npm-requestretry-delaystrategey] to see how to define your own delay strategy.
  * **strip$$Properties:** strips the $$-properties from the payload. The default is true.
  * **logging:** logs the response body if the status code >= 400 to the console for any value. If the value is 'debug' the request url will also be logged to the console.

#### examples ####

```javascript
  const respsOfTeam = await sriClient.getAll('/responsibilities', {organisationalUnit: '/organisationalunits/eb745d58-b818-4569-a06e-68733fe2e5b3'}, {logging: 'debug'});
  const personHrefs = respsOfTeam.map(resp => resp.person.href);
  const persons = await vskoApi.getAllHrefs(personHrefs, '/persons/batch', undefined, {asMap: true});
  // persons is a map with as key the href of a person and as value the resource of that person.

  const externalIdentifierWithOrganisationalUnitExpanded = await sriClient.get(
    '/organisationalunits/externalidentifiers',
    {value: '032557'},
    {
      expand: [
        'organisationalUnit.$$contactDetails.phone.organisationalUnit', // this is circular so not very usefull, but this shows you can expand as deep as you want to
        'organisationalUnit.$$contactDetails.email'
      ]
    });

  const organisationalUnitWithInstitutionNumberIncluded = await sriClient.get('/organisationalunits/a2c36c96-a3a4-11e3-ace8-005056872b95', undefined, {include: {
    alias: '$$institutionNumber',
    href: '/organisationalunits/externalidentifiers',
    filters: {type: 'INSTITUTION_NUMBER'},
    reference: 'organisationalUnit',
    singleton: true
  }});

  const personWithResponsibilitiesIncluded = await sriClient.get('/persons/94417de5-840c-4df4-a10d-fe30683d99e1', undefined, {include: {
    alias: '$$responsibilities',
    href: '/responsibilities',
    reference: {
      property: 'person',
      parameterName: 'personIn'
    },
    expand: ['organisationalUnit']
  }});

  // give all the responsibilies of an organisationalunit (this is the team)
  // and expand the person (client side).
  // Include for this person all his responsibilities
  // and expand the organisationalunits of these responsibilities
  sriClient.getAll('/responsibilies',
    {
      organisationalunit: '/organisationalunits/eb745d58-b818-4569-a06e-68733fe2e5b3',
      expand: 'position' // expand position server side
    },
    {
      expand: [{
        property: 'person', // expand all person properties client side,
        required: false,
        include: [{
          alias: '$$responsibilities', // A $$responsibilities property will be added to every resource in the resultset
          href: '/responsibilities',
          reference: {
            property: 'person',
            parameterName: 'personIn',
          }
          expand: ['organisationalUnit'], // client side expansion of properties within the included resources
        }]
      }]
    }
  );
  // for each responsibility in the resultset, the expanded person will have an added property $$responsibilities,
  // which is an array of all the responsibilities that reference this person.
  // Within these last responsibilities the organisationalUnit will be expanded
  ```

### ng-sri-client ###

```javascript
@gunther add here how to use the module.
```

In a future module we might add an Oauth Interceptor module to support authentication to an Oauth Server.
On every call to the baseUrl configured in the configuration it will make sure that a bearer token is added for authentication.
If the token is expired it will take care of getting a new one. If you are not or no longer logged in it will redirect you to the login page.

#### initialisation ####

ng-sri-client requires to specify a baseUrl and the oauth parameters.
Call the sriClientConfiguration service and call sriClientConfiguration.set(newConfiguration) to initialise the configuration for the module.

The possible properties are:

* baseUrl: the default baseUrl for all the api calls.
* oauth: add here the vsko-oauth-configuration.json object that is generated by the deploy process. If not present or false, there will be no oauth integration.
* logging: For every request logs the response body if the status code >= 400 to the console for any value. If the value is 'debug' the request url will also be logged to the console.

### node-sri-client ###

This module is build upon [requestretry][npm-requestretry] which itself is build upon [request][npm-request].
Here is an example how to use the module.

```javascript
const configuration = {
  baseUrl: 'https://api.katholiekonderwijs.vlaanderen'
}

const api = require('sri-client/node-sri-client')(configuration)

let secondarySchools = await api.get('/schools', {educationLevels: 'SECUNDAIR'});
```

this configuration can have te following properties:

* baseUrl: the default baseUrl for all the api calls.
* username: username for basic authentication calls.
* password: password for basic authentication calls.
* headers: each request will have the headers specified added in the request header.
* accessToken: an object with properties name and value. Each request will have a request header added with the given name and value. This is added to the headers if they are specified.

#### error handling ####

If the response is different from status code 200 or 201 or there is no response the response is rejected and an error object is returned of type SriClientError.
So you can catch errors coming from the sri client and catch http error by filtering on  this error, for example:

```javascript
// create a batch array
try {
  await api.put('/batch', batch);
} catch (error) {
  if(error instanceof api.SriClientError) {
    console.error(util.inspect(error.body, {depth:7}));
    console.error(error.stack);
  } else {
    console.error(error.stack);
  }
}
```

An SriClientError has the following properties:

* status: the status code of the response (null if there was no response)
* body: payload of the response.
* getResponseHeader(): function that returns the whole responseHeader [is not working yet]
* stack: stack trace that leads back to where the api was called from in the code

## common-utils ##

This is a library with common utility functions

#### usage ####
```javascript
const commonUtils = require('sri-client/common-utils');
```

#### interface ####

* **generateUUID():** returns a generated uuid of version 4.
* **splitPermalink(permalink):** returns an object with properties key (the key of the permalink) and path (the path of the permalink without the key)
* **getKeyFromPermalink(permalink):** returns the key within the permalink
* **getPathFromPermalink(permalink):** returns the pathe of the permalink without the key
* **strip$$Properties(object):** returns a copy of the object with all the properties that start with '$$' removed
* **strip$$PropertiesFromBatch(batchArray):** returns a copy of the batchArray with all the properties of the body in the batch objects that start with '$$' removed

## date-utils ##

This is a library with utility functions for string dates as specified in the sri api in the format yyyy-MM-dd.
So if we talk about a date as a string it is in that format.
If a date as a string is null or undefined it is interpreted as infinitely in the future.

#### usage ####
```javascript
const dateUtils = require('sri-client/date-utils');
```

#### interface ####

* **getNow():** returns the current date as a string
* **setNow(dateString):** sets now to another date for the whole library. From now on getNow() will return this date.
* **toString(date):** return the javascript date as a string
* **parse(dateString):** returns the dateString as a javascript date
* **stripTime(isoDateString):** returns the received isoDateString without the time section. YYYY-MM-DDTHH:mm:ss.sssZ -> YYYY-MM-DD
* **isBeforeOrEqual(a,b):** returns true if a is before or on the same day as b, where a and b are dates as strings
* **isAfterOrEqual(a,b):** returns true if a is after or on the same day as b, where a and b are dates as strings
* **isBefore(a,b):** returns true if a is strictly before b, where a and b are dates as strings
* **isAfter(a,b):** returns true if a is strictly after b, where a and b are dates as strings
* **getFirst(arrayOfDateStrings):** returns the date that is first in time from arrayOfDateStrings
* **getLast(arrayOfDateStrings):** returns the date that is last in time from arrayOfDateStrings
* **isOverlapping(a, b):** returns true if there is an overlapping period between a en b where a and b are objects with a property startDate and endDate (which can be null/undefined)
* **isCovering(a, b):** returns true if the period of b is completely within the period of a.
* **isConsecutive(a, b):**: returns true if the periods of a and b are strictly following each other in any order. So b starts on the same date as a ends or a starts on the same date as b ends.
* **isConsecutiveWithOneDayInBetween(a, b):**: returns true if the periods of a and b are strictly following each other in any order with one day in between. So b starts the day after a ends or a starts the day after b ends.
* **getStartOfSchoolYear(dateString):** returns the first of september before datestring (the first of september before getNow() if dateString is null),
* **getEndOfSchoolYear(dateString):** returns the first of september after dateString (the first of september after getNow() if dateString is null),
* **getPreviousDay(dateString):** returns the day before dateString as a string
* **getNextDay(dateString):** returns the day after dateString as a string
* **getActiveResources(arrayOfResources, referenceDateString):** returns a new array with only the resources that are active on the referenceDateString (getNow() if dateString is null) from array,
which is an array of resources with a period. array can also be an array of hrefs that is expanded.
* **getNonAbolishedResources(arrayOfResources, referenceDateString):**  returns a new array with only the resources that are not abolished on the referenceDateString (getNow() if dateString is null) from array,
which is an array of resources with a period. array can also be an array of hrefs that is expanded.
* **manageDateChanges(resource, options, sriClient):** manages the date changes for the given resource using the given sriClient. The options are:
  * oldStartDate: the old startDate of the resource before it was updated.
  * oldEndDate: the old endDate of the resource before it was updated.
  * batch: a batch array. All references to this resource that should be updated as well will be added to this batch array. If a depending resource is already present in the batch, it will update the body of that batch element.
  * properties: an array of strings containing the property names of the resource that have a period which should be updated together with the main period of the resource.
  * references: an array of objects (or one object) with configuration to find the other resources referencing this resource and by consequence should have their period updated as well. The configuration has the following properties:
    * alias: not required but if you add an alias, the function will return an object with the given alias as a property which contains the referencing resources that are updates as well. If there was no period change the returned object is null.
    * href: the path on which the referencing resources can be found.
    * property: the property name/parameter name of the referencing resource pointing to the given resource.
    * commonReference: If a referencing resource is not referencing directly to the given resource but they are related to each other because they are referencing to the same resource. For example for the educationalprogrammedetails/locations which have to keep up to date with the organisationalunits/locations referencing the same phisicalLocation (and the same organisational unit).
    * parameters: optional parameters to filter on the references.
    * options: an options object that will be passed on as a parameter to the sriClient.getAll function.
If an error occurs when hanling a periodic reference to the given resource, a DateError is thrown with two properties, a message and the periodic.

## address-utils ##

This is a library with utility functions for address objects as specified in the sri api.

#### usage ####
```javascript
const addressUtils = require('sri-client/address-utils');
```

#### interface ####

* **isSameHouseNumberAndMailbox(a, b):** returns true if sri address a and sri address b have the same mailboxNumber and houseNumber. The match is case insensitive and ingores white spaces and underscores.
* **isSameStreet(a, b):** returns true if sri address a and sri address b are the same streets. This means a match on  the street name in the same city. If both addresses have a streetHref a match is done based on this reference because it is a reference to the same street, independent of how it is spelled. Otherwise A match on street name is case insensitive and takes into account that parts of the name are abbreviated with a dot. For example 'F. Lintsstraat' matches with 'Frederik lintsstraat'.
* **addSubCityHref(sriAddress, api) [async]:** adds a subCityHref reference to the sriAddress. api is an instance of an sri-client library (can be both the ng-sri-client or the node-sri-client)
* **addStreetHref(sriAddress, api) [async]:** adds a streetHref reference to the sriAddress. api is an instance of an sri-client library (can be both the ng-sri-client or the node-sri-client)

### Questions ###

Mail to matthias.snellings@katholiekonderwijs.vlaanderen, gunther.claes@katholiekonderwijs.vlaanderen.

[sri-documentation]: https://github.com/dimitrydhondt/sri
[sri4node-project]: https://github.com/katholiek-onderwijs-vlaanderen/sri4node
[npm-requestretry]: https://www.npmjs.com/package/requestretry
[npm-requestretry-strategy]: https://www.npmjs.com/package/requestretry#how-to-define-your-own-retry-strategy
[npm-requestretry-delaystrategey]: https://www.npmjs.com/package/requestretry#how-to-define-your-own-delay-strategy
[npm-request]: https://www.npmjs.com/package/request