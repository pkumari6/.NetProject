#Couchbase .NET Client SDK Tutorial
This tutorial bridges the gap between simple and advanced Couchbase concepts by walking through a complete web application using these Couchbase .NET client library, covering N1QL and key-value set/get operations.

The full source code for the tutorial is available on GitHub: [github.com/couchbaselabs/try-cb-dotnet](https://github.com/couchbaselabs/try-cb-dotnet).

This tutorial makes use of the `travel-sample` data-set that comes with Couchbase Server. 

The HTML/JavaScript code that generates the web application is provided with the source code but this tutorial does not explain that side of the implementation. 

After completing this tutorial you will have learned how to:

* install and configure the Couchbase .NET Client
* bootstrap the .NET Client
* use LINQ with N1QL
* use Couchbase in your own projects.
* use recommended practices for working with Couchbase with .NET
* apply basic knowledge of using Couchbase Server from within your .NET applications.

![Application Screen shot](content/images/Screen Shot 2015-11-05 at 11.49.06.png)

##Prerequisites and Setup
You will need to have the following available/installed:

* [Visual Studio 2015](https://www.visualstudio.com/) or newer (The source code is created using VS 2015 Professional)
* Windows 8.1 or higher (to be able to install and run Visual Studio 2015)
* Although not a requirement, we recommend you have a Git client for easy source code browsing and making it easy to switch between branches (tutorial steps are split using branches).


##Installing Couchbase Server
First things first: we need to install Couchbase Server! You can chose to install it locally on your developer machine or remotely. In this tutorial we will assume that Couchbase Server (4.0 or greater) is installed locally on your development machine alongside the website that we will create.

Download Couchbase Server and follow the [instructions](http://developer.couchbase.com/documentation/server/current/getting-started/installing.html) for your platform to complete the installation. 

If this is your first time setting up Couchbase Server, this [detailed guide](http://developer.couchbase.com/documentation/server/current/install/init-setup.html#topic12527) will help you through all the steps and explain the different options. 

**Important!** As you follow the download instructions and setup wizard, make sure you keep all the services (data, query, and index) selected and remember to install the sample bucket named `travel-sample` (introduced in CB 4.0). `travel-sample` is the dataset that we will use throughout this tutorial.

![Select all services](content/images/setup-01.png)

>TIP: 
	If you already have Couchbase Server installed but did not install the travel-sample bucket!
	
>Open the Couchbase Web Console and select 'Settings' -> 'Sample Buckets' and select the `travel-sample` check box, and then click Create. 
	 
>A notification box in the upper-right corner will appear and show the progress. When it disappears the bucket is ready to use.
	
##Getting ready
###Understanding the source code repository 
The source code is split into branches. Every branch represents a step in this tutorial and every step builds on the previous step. The final result is in the `master` branch.

* `tutorial-part-1` is the most simple skeleton that can compile and show a UI. It's not possible to navigate the app yet.
* `tutorial-part-2` is the result of part 1 and returns static content to allow the user to browse the site. It returns only static data.
* `tutorial-part-3` is the result of part 2 and adds queries and live data to the site. It's now possible to navigate the site and get actual data back served from Couchbase Server.
* `tutorial-part-4` is the result of part 3 and adds user login and password storage to the site.
* `tutorial-part-5` is the result of part 4 and shows a few extra options in the Couchbase .NET SDK, such as LINQ support.
* `master` is the final result after refactoring part 5.

###A quick note on the source itself
This source code is split up into three separate parts, shown in the following diagram.
 
1. Couchbase: The red box is the Couchbase Server Cluster serving our data
2. Backend: The green box represents the API server, responsible for serving the data to the UI. In this case the backend is built with C# and Web API, but could be implemented in any language. We currently have tutorials for: [Java](https://github.com/couchbaselabs/try-cb-java), [Node.js](https://github.com/couchbaselabs/try-cb-nodejs) and [.net - this tutorial](https://github.com/couchbaselabs/try-cb-dotnet)
3. Frontend: The blue box symbolises the UI itself. This is the part of the application that surfaces the content to the user. It's all static HTML and JavaScript built to consume data from the API. We are not going to change a single line in the UI. 

![app diagram](content/images/Screen Shot 2015-11-05 at 12.00.18.png)

This application architecture gives a very clean separation of concerns and would allow us to change the various parts in the application architecture without influencing the other parts. This is, of course, only true if we maintain the REST API methods and responses.

In fact, this allows us to change the backend API without touching the front end code. 
Therefore, if you take a look at the Java version [`try-cb-java`](https://github.com/couchbaselabs/try-cb-java) you quickly learn that the UI is the same: it's only the backend that has changed.

In the .NET implementation of the backend we use WEB API, as this is probably the most popular, flexible and easy way to implement a REST API endpoint in .NET.  


###Get set up for the tutorial 
To get properly set up for the tutorial, follow these steps:

* git clone https://github.com/couchbaselabs/try-cb-dotnet.git or download the source
* If you did not install Couchbase Server on `localhost` and want to connect to a remote Couchbase Server, change the `couchbaseServer` key in `web.config`. You can also change the username and password for Couchbase Server.
* Use Visual Studio 2015 to open the solution file `try-cb-dotnet.sln` in `try-cb-dotnet/src`
* The solution is configured to restore all missing `nuget`packages on every build. Therefore the only thing missing now is to build and run the solution.

>Note:
>
>Restoring the missing NuGet Packages can take some time and is also influenced by your network speed.

#Tutorial Step 1 - 5
 
##Step 1 - Understand ASP.NET Web API 2 and .NET 
The first part of this tutorial is not about how to use the .NET Client for Couchbase Server. 

The focus in part 1 is to show how ASP.NET Web API works and to emphasise that with Web API we have an option to work with and return JSON from the REST endpoints. 

It's important to understand that the main task of the backend REST API is to return JSON. This is an important concept to understand that will greatly help you work with ASP.NET's Web API.
 
Implementing the Web API methods (REST endpoints) is all about returning the right JSON. Couchbase stores all documents in JSON, making it a good data store for a REST API.
 
*If you already feel comfortable working with ASP.NET Web API you can skip this step and go directly to step 2. In step 2 you will learn how to use Couchbase and the Couchbase .NET Client in your .NET projects.*

In this step you will update all Web API methods to return static JSON (string values). This will allow you to run and browse the web application and get an understanding of how the code is organised.

###Step 1.1 - Implement login 

**Where:** `UserController.cs` -> **method:** `Login(string password, string user)`

**Goals:** Return static JSON to learn how Web API works.

**Relevant Documentation Topics:** 

 * [ASP.NET Web API 2](http://www.asp.net/web-api/overview/getting-started-with-aspnet-web-api/tutorial-your-first-web-api)
 * [JWT Token](http://jwt.io/)

**Task:**

`Login(string password, string user)` is a Web API method called using JavaScript from the static HTML page `index.html`. 

The JavaScript used in the static HTML page expects this "Login" web api call to return a JavaScript Web Token (JWT) "success" status code token. Returning the "success" token will log the user in and redirect the user to the flight booking page. 

The JWT token is used to reference and store data about the user's trips/bookings and login credentials.

The JWT response should be in a JSON format like this:

```JSON
[{"success":"eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyIjoiZ3Vlc3QiLCJpYXQiOjE0NDE4Njk5NTR9.5jPBtqralE3W3LPtS - j3MClTjwP9ggXSCDt3 - zZOoKU"}]
```

Implement the method to return a "success" JWT token allowing the user to log in.

Later we will implement a JWT token issuer and store user data in Couchbase for later look-up.
The token is created for the user:

>Note: 
>
>The login credentials for this JWT token is:
>
>username: guest
>
>password: guest

**Solution:**

Update the Login method to return the JWT token value:

```C#
[HttpGet]
[ActionName("Login")]
public object Login(string password, string user)
{
   
    return new { success = 	"eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9." +
            "eyJ1c2VyIjoiZ3Vlc3QiLCJpYXQiOjE0NDE4Njk5NTR9.5jPBtqralE3W3LPtS - " +
            "j3MClTjwP9ggXSCDt3 - zZOoKU" };
}
```    
        
###Step 1.2

**Where:** `UserController.cs` -> **method:** `Login([FromBody] UserModel user)`

**Goals:** Return static JSON to learn how ASP.NET Web API works.

**Relevant Documentation Topics:** [ASP.NET Web API 2](http://www.asp.net/web-api/overview/getting-started-with-aspnet-web-api/tutorial-your-first-web-api)


**Task:**
This task is essentially the same as `1.1`.

`Login(string password, string user)` is a Web API method called using JavaScript from the static HTML page `index.html`. 

The JavaScript used in the static HTML page expects this "Login" web api call to return a JavaScript Web Token (JWT) "success" status code token. Returning the "success" token will login the user and redirect the user to the flight booking page. 

The JWT token is used to reference and store data about the user's trips/bookings and login credentials.

The JWT response should be in a JSON format like this:

```JSON
[{"success":"eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyIjoiZ3Vlc3QiLCJpYXQiOjE0NDE4Njk5NTR9.5jPBtqralE3W3LPtS - j3MClTjwP9ggXSCDt3 - zZOoKU"}]
```

Implement the method to return a "success" JWT token allowing the user to login.

Later we will implement a JWT token issuer and store user data in Couchbase for later look-up.
The token is created for the user:

>Note: The login credentials for this JWT token is:
>
>username: guest
>
>password: guest

**Solution:**

```C#
[HttpPost]
[ActionName("Login")]
public object CreateLogin([FromBody] UserModel user)
{        
	return new { success = 	"eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9." +
	"eyJ1c2VyIjoiZ3Vlc3QiLCJpYXQiOjE0NDE4Njk5NTR9.5jPBtqralE3W3LPtS - " +
	"j3MClTjwP9ggXSCDt3 - zZOoKU" };
}
```

###Step 1.3

**Where:** `UserController.cs` -> **method:** `Flights(string token)`

**Goals:** Return static JSON to learn how WEB API works.

**Relevant Documentation Topics:** [ASP.NET WEB API 2](http://www.asp.net/web-api/overview/getting-started-with-aspnet-web-api/tutorial-your-first-web-api)

**Task:**

This is a Web API call, a method that is called from the static HTML (index.html).
The JavaScript in the static HTML expects this "flights" Web API call to return
all bookings made by the logged in user. 

The JWT token is used to look-up the user and find all bookings.

In this "fake" implementation we are not going to use the JWT Token, but instead return a static list of bookings. The list should be the same for all users.

The response should be in a JSON format like this:

Bookings:

```JSON
[{"_type":"Flight","_id":"d500a3d1-2cca-43a5-8a66-f11828a35969","name":"American Airlines","flight":"AA344","date":"09/10/2015","sourceairport":"SFO","destinationairport":"LAX","bookedon":"1441881827622"},{"_type":"Flight","_id":"bf676b0d-e63b-4ff6-aade-7ac1c182b3de","name":"American Airlines","flight":"AA787","date":"09/11/2015","sourceairport":"LAX","destinationairport":"SFO","bookedon":"1441881827623"},{"_type":"Flight","_id":"f0099c24-3ad4-482e-8352-704f9cbf1a43","name":"American Airlines","flight":"AA550","date":"09/10/2015","sourceairport":"SFO","destinationairport":"LAX","bookedon":"1441881827623"}]
```
            
Implement the method to return the "fake" bookings for all users.
Later we will look-up actual bookings using the JWT token, but for now a static list is what we need.

>Hint: 
>
>To simulate more than one booking you can return the same booking multiple times in a list, re-using the sample JSON above.

**Solution:**

```C#
[HttpGet]
[ActionName("flights")]
public object Flights(string token)
{
       
	return new List<dynamic>
	{
		new {
		_type="Flight",_id="f0099c24-3ad4-482e-8352-704f9cbf1a43",name="American Airlines",flight="AA550",date="09/10/2015",sourceairport="SFO",destinationairport="LAX",bookedon=1441881827623},
            new {_type="Flight",_id="f0099c24-3ad4-482e-8352-704f9cbf1a43",name="American Airlines",flight="AA550",date="09/10/2015",sourceairport="SFO",destinationairport="LAX",bookedon=1441881827623},
            new {_type="Flight",_id="f0099c24-3ad4-482e-8352-704f9cbf1a43",name="American Airlines",flight="AA550",date="09/10/2015",sourceairport="SFO",destinationairport="LAX",bookedon=1441881827623},
        };
}    
```    

###Step 1.4

**Where:** `UserController.cs` -> **method:** `BookFlights([FromBody] dynamic request)`

**Goals:** Return static JSON to learn how WEB API works.

**Relevant Documentation Topics:** [ASP.NET WEB API 2](http://www.asp.net/web-api/overview/getting-started-with-aspnet-web-api/tutorial-your-first-web-api)

**Task:**    
This is a Web API call, a method that is called from the static HTML (index.html).
The JavaScript in the static HTML expects the call to the "flights" Web API method to save the flight bookings for a user. The bookings are saved in a bookings document.
 
The JWT token is used as a key to the user's bookings document, this way we can use the JWT token to look-up all bookings for a user.

In this "fake" implementation we are not going to use the JWT Token, nor store any data about the bookings.

Instead we return a static value to indicate that the booking was successful, we simply simulate the creation of the bookings document for the user.

The response should be in a JSON format like this:

Bookings:

```JSON
{"added":3}`
```
 
Implement the Web API method to return the "fake" `booking success` for the guest user.

Later we will revisit this method and updated it to store bookings using the JWT token, but for now a static response is what we need.

**Solution:**

```C#
[HttpPost]
[ActionName("flights")]
public object BookFlights([FromBody] dynamic request)
{
	return new { added = 3 };
}
```    

###Step 1.5

**Where:** `FlightPathController.cs` -> **method:** `FindAll(string from, DateTime leave, string to, string token)`

**Goals:** Return static JSON to learn how WEB API works.

**Relevant Documentation Topics:** [ASP.NET WEB API 2](http://www.asp.net/web-api/overview/getting-started-with-aspnet-web-api/tutorial-your-first-web-api)

**Task:**  

Task:
This is a Web API call, a method that is called from the static html page `index.html`.

JavaScript is used in the static `index.html` page to call the Web API and the expected value returned from the `findAll` Web API call is "trip" data in a JSON format like this:

Round trip: 

```JSON
[{"destinationairport":"SFO","equipment":"738","flight":"AA907","id":5746,"name":"American Airlines","sourceairport":"LAX","utc":"00:29:00","flighttime":1,"price":53},{"destinationairport":"SFO","equipment":"738","flight":"AA787","id":5746,"name":"American Airlines","sourceairport":"LAX","utc":"19:06:00","flighttime":1,"price":45},{"destinationairport":"SFO","equipment":"738","flight":"AA279","id":5746,"name":"American Airlines","sourceairport":"LAX","utc":"04:54:00","flighttime":1,"price":52},{"destinationairport":"SFO","equipment":"E75","flight":"DL856","id":21085,"name":"Delta Air Lines","sourceairport":"LAX","utc":"20:08:00","flighttime":1,"price":47},{"destinationairport":"SFO","equipment":"E75","flight":"DL273","id":21085,"name":"Delta Air Lines","sourceairport":"LAX","utc":"14:14:00","flighttime":1,"price":48},{"destinationairport":"SFO","equipment":"73W 73C 733","flight":"WN543","id":63986,"name":"Southwest Airlines","sourceairport":"LAX","utc":"22:16:00","flighttime":1,"price":44},{"destinationairport":"SFO","equipment":"73W 73C 733","flight":"WN828","id":63986,"name":"Southwest Airlines","sourceairport":"LAX","utc":"04:35:00","flighttime":1,"price":43},{"destinationairport":"SFO","equipment":"738","flight":"US086","id":59532,"name":"US Airways","sourceairport":"LAX","utc":"15:06:00","flighttime":1,"price":46},{"destinationairport":"SFO","equipment":"738","flight":"US150","id":59532,"name":"US Airways","sourceairport":"LAX","utc":"15:44:00","flighttime":1,"price":47},{"destinationairport":"SFO","equipment":"738","flight":"US437","id":59532,"name":"US Airways","sourceairport":"LAX","utc":"23:42:00","flighttime":1,"price":52},{"destinationairport":"SFO","equipment":"739 752 753 319 320 738","flight":"UA666","id":57010,"name":"United Airlines","sourceairport":"LAX","utc":"05:11:00","flighttime":1,"price":44},{"destinationairport":"SFO","equipment":"739 752 753 319 320 738","flight":"UA978","id":57010,"name":"United Airlines","sourceairport":"LAX","utc":"19:50:00","flighttime":1,"price":53},{"destinationairport":"SFO","equipment":"739 752 753 319 320 738","flight":"UA123","id":57010,"name":"United Airlines","sourceairport":"LAX","utc":"21:13:00","flighttime":1,"price":49},{"destinationairport":"SFO","equipment":"320 319","flight":"VX929","id":62018,"name":"Virgin America","sourceairport":"LAX","utc":"00:39:00","flighttime":1,"price":49},{"destinationairport":"SFO","equipment":"320 319","flight":"VX351","id":62018,"name":"Virgin America","sourceairport":"LAX","utc":"01:37:00","flighttime":1,"price":49},{"destinationairport":"SFO","equipment":"320 319","flight":"VX703","id":62018,"name":"Virgin America","sourceairport":"LAX","utc":"05:01:00","flighttime":1,"price":47},{"destinationairport":"SFO","equipment":"320 319","flight":"VX743","id":62018,"name":"Virgin America","sourceairport":"LAX","utc":"10:36:00","flighttime":1,"price":53},{"destinationairport":"SFO","equipment":"320 319","flight":"VX301","id":62018,"name":"Virgin America","sourceairport":"LAX","utc":"01:32:00","flighttime":1,"price":49}]
```
            
One way trip:

```JSON
[{"destinationairport":"SFO","equipment":"738","flight":"AA787","id":5746,"name":"American Airlines","sourceairport":"LAX","utc":"19:06:00","flighttime":1,"price":48},{"destinationairport":"SFO","equipment":"738","flight":"AA279","id":5746,"name":"American Airlines","sourceairport":"LAX","utc":"04:54:00","flighttime":1,"price":49},{"destinationairport":"SFO","equipment":"738","flight":"AA907","id":5746,"name":"American Airlines","sourceairport":"LAX","utc":"00:29:00","flighttime":1,"price":51},{"destinationairport":"SFO","equipment":"E75","flight":"DL273","id":21085,"name":"Delta Air Lines","sourceairport":"LAX","utc":"14:14:00","flighttime":1,"price":51},{"destinationairport":"SFO","equipment":"E75","flight":"DL856","id":21085,"name":"Delta Air Lines","sourceairport":"LAX","utc":"20:08:00","flighttime":1,"price":53},{"destinationairport":"SFO","equipment":"73W 73C 733","flight":"WN543","id":63986,"name":"Southwest Airlines","sourceairport":"LAX","utc":"22:16:00","flighttime":1,"price":50},{"destinationairport":"SFO","equipment":"73W 73C 733","flight":"WN828","id":63986,"name":"Southwest Airlines","sourceairport":"LAX","utc":"04:35:00","flighttime":1,"price":53},{"destinationairport":"SFO","equipment":"738","flight":"US086","id":59532,"name":"US Airways","sourceairport":"LAX","utc":"15:06:00","flighttime":1,"price":53},{"destinationairport":"SFO","equipment":"738","flight":"US150","id":59532,"name":"US Airways","sourceairport":"LAX","utc":"15:44:00","flighttime":1,"price":43},{"destinationairport":"SFO","equipment":"738","flight":"US437","id":59532,"name":"US Airways","sourceairport":"LAX","utc":"23:42:00","flighttime":1,"price":48},{"destinationairport":"SFO","equipment":"739 752 753 319 320 738","flight":"UA978","id":57010,"name":"United Airlines","sourceairport":"LAX","utc":"19:50:00","flighttime":1,"price":48},{"destinationairport":"SFO","equipment":"739 752 753 319 320 738","flight":"UA666","id":57010,"name":"United Airlines","sourceairport":"LAX","utc":"05:11:00","flighttime":1,"price":51},{"destinationairport":"SFO","equipment":"739 752 753 319 320 738","flight":"UA123","id":57010,"name":"United Airlines","sourceairport":"LAX","utc":"21:13:00","flighttime":1,"price":50},{"destinationairport":"SFO","equipment":"320 319","flight":"VX743","id":62018,"name":"Virgin America","sourceairport":"LAX","utc":"10:36:00","flighttime":1,"price":48},{"destinationairport":"SFO","equipment":"320 319","flight":"VX703","id":62018,"name":"Virgin America","sourceairport":"LAX","utc":"05:01:00","flighttime":1,"price":45},{"destinationairport":"SFO","equipment":"320 319","flight":"VX301","id":62018,"name":"Virgin America","sourceairport":"LAX","utc":"01:32:00","flighttime":1,"price":49},{"destinationairport":"SFO","equipment":"320 319","flight":"VX929","id":62018,"name":"Virgin America","sourceairport":"LAX","utc":"00:39:00","flighttime":1,"price":44},{"destinationairport":"SFO","equipment":"320 319","flight":"VX351","id":62018,"name":"Virgin America","sourceairport":"LAX","utc":"01:37:00","flighttime":1,"price":46}]
```

As shown above, the "trip" data is spilt up in two: one way and round trip (return trip). 

Implement the method to return a "round trip" from a destination and source airport and back.

Later we re-vist this Web API method and update it to use data from Couchbase to do the look-up, but for now a "constant" is returned.          
            
**Solution:**    

```C#
[HttpGet]
[ActionName("findAll")]
public object FindAll(string from, DateTime leave, string to, string token)
{
	return new List<dynamic>
    {
    	new { destinationairport="SFO",equipment=738,flight="AA907",id=5746,name="American Airlines",sourceairport="LAX",utc="00:29:00",flighttime=1,price=53},
    	new { destinationairport="SFO",equipment=738,flight="AA907",id=5746,name="American Airlines",sourceairport="LAX",utc="00:29:00",flighttime=1,price=53},
    	new { destinationairport="SFO",equipment=738,flight="AA907",id=5746,name="American Airlines",sourceairport="LAX",utc="00:29:00",flighttime=1,price=53}
    };
}
```

###Step 1.6

**Where:** `AirportController.cs` -> **method:** `FindAll(string search, string token)`

**Goals:** Return static JSON to learn how WEB API works.

**Relevant Documentation Topics:** [ASP.NET WEB API 2](http://www.asp.net/web-api/overview/getting-started-with-aspnet-web-api/tutorial-your-first-web-api)

**Task:**    

This is a Web API call, a method that is called from the static html (index.html).
The JavaScript in the static html expects this `findAll` Web API call to return a
"airportname", based on the input `search` parameter.

The return data used for this static implementation should look as follows:

```JSON
[{"airportname":"San Francisco Intl"}]
```

Implement the method to return a single airport name.

Later we will use Couchbase to do the look-up, but for now a constant value is returned.

**Solution:**

```C#
[HttpGet]
[ActionName("findAll")]
public object FindAll(string search, string token)
{
    return new List<dynamic>()
    {
        new {airportname = "San Francisco Intl"}
    };
}
```
    
###Step 1 - Summary
If done correctly all Web API methods now return a static JSON value. This should enable you to run and browse the application. 

All data is static but never the less it "works". In Step 2 we will update the static JSON returned in the Web API method to return actual data from Couchbase Server.    

##Step 2 - Understand the Couchbase .NET SDK & N1QL
In this step we will update all ASP.NET Web API methods to return data from Couchbase Server. 

This is the first step that uses Couchbase and therefore we need to bootstrap by:

 * adding a reference to the Couchbase Client
 * adding a reference to the LINQ extensions
 * configuring the Couchbase Client. 

###Step 2.0 - Referencing and bootstrapping the Couchbase client for .NET.

**Where:** `Solution` (this is a solution-wide update)

**Goals:** Add a reference to: [CouchbaseNetClient](https://www.nuget.org/packages/CouchbaseNetClient) and [Linq2Couchbase](https://www.nuget.org/packages/Linq2Couchbase) the later is the LINQ to N1QL extensions. 

When the references are in place we need to bootstrap (configuration and initialisation) the Couchbase .NET SDK and make it globally available to the application.

**Relevant Documentation Topics:** 

* [N1QL intro](http://developer.couchbase.com/documentation/server/4.0/n1ql/n1ql-intro/data-access-using-n1ql.html)
* [Couchbase .NET Client - GitHub](https://github.com/couchbase/couchbase-net-client)
* [Couchbase .NET Client - docs](http://developer.couchbase.com/documentation/server/4.0/sdks/dotnet-2.2/getting-started.html)
* [Linq2Couchbase - GitHub](https://github.com/couchbaselabs/Linq2Couchbase)
* [Hello World - Couchbase .NET](http://developer.couchbase.com/documentation/server/4.0/sdks/dotnet-2.2/hello-couchbase.html)

### 2.0 - Task 1: Referencing Couchbase Client and LINQ to N1QL Extensions
For every release of the Couchbase .NET Client a matching NuGet package with the binaries is released to the [NuGet Gallery](https://www.nuget.org/packages). 

If you are not familiar with NuGet, it’s the official and most widely supported package manager for Microsoft Visual Studio and .NET in general. NuGet is a centralised repository for package authors and consumers, and it also defines a suite of tools for authoring, publishing and consuming packages from various vendors and authors.

Using Visual Studio 2015 or later, follow these steps to get started with the Couchbase .NET SDK:

1. From the IDE, right-click the solution/project to which you want to add the dependency.

	![Add NuGet Ref](content/images/Screen Shot 2015-11-18 at 08.23.42.png)


2. In the context menu, click `Manage NuGet Packages for ...`. The NuGet package manager view opens.

	![NuGet view](content/images/Screen Shot 2015-11-18 at 08.30.52.png)

3. In the search box at the top right-hand side of the dialog, type `CouchbaseNetClient` and press 'Enter' on your keyboard to start the search. 

	![NuGet install CB Client](content/images/Screen Shot 2015-11-18 at 08.34.20.png)

4. In the search results, select the 'CouchbaseNetClient' package and then click Install. The Visual Studio Output window will show progress and concluded with a 'Finished'.

	![NuGet install progress](content/images/Screen Shot 2015-11-18 at 08.36.36.png)

5. Repeat step 3-4 to install `Linq2Couchbase`.
	
	![NuGet installed packages](content/images/Screen Shot 2015-11-18 at 08.41.21.png)	
	
6. Confirm that both NuGet packages have been successfully installed.
	Using the NuGet view, search for `Couchbase` and set Filter to `Installed` as shown above.
	Confirm that both `Linq2Couchbase` and `CouchbaseNetClient` are installed.

	That’s it! NuGet has pulled in all required dependencies and references required for the Couchbase Client and LINQ to N1QL extension. 


###2.0 - Task 2: Bootstrapping Couchbase Client 
We need to inform the .NET Client where to find the Couchbase Server and what buckets to use.

The Couchbase Client includes a helper class called `ClusterHelper`. This class is a singleton that can be shared globally in the application and should be kept alive for the lifetime of the application. The benefit of using `ClusterHelper` is that resources are shared across the the application and thereby setup and teardown is not done unless explicitly needed. 

It is recommended by Couchbase to use `ClusterHelper` when working with the .NET Client and part of the best practices. 

In this tutorial the application at hand is a web application and therefore it's most convenient to initialise the Couchbase Client in `Global.asax.cs` as this is the main entry point and run on application start. Initialising the Couchbase .NET Client here will ensure that it is initialised before any other code is called.

Using Visual Studio 2015 or later, follow these steps to bootstrap the Couchbase .NET Client:

1. Create a new file in the folder 'App_Start' called 'CouchbaseConfig.cs'.
	
	![Add new file 1](content/images/Screen Shot 2015-11-18 at 09.28.04.png)

	![Add new file 2](content/images/Screen Shot 2015-11-18 at 09.29.34.png)

2. Replace the content of `CouchbaseConfig.cs` with this code snippet:
	
	```C#
	using System;
	using System.Collections.Generic;
	using System.Linq;
	using System.Text;
	using System.Threading.Tasks;
	using Couchbase;
	using Couchbase.Configuration.Client;
	
	namespace try_cb_dotnet.App_Start
	{
		public static class CouchbaseConfig
		{
    		public static void Initialize()
    		{
        		var config = new ClientConfiguration();
        		config.BucketConfigs.Clear();
	
        		config.Servers = new List<Uri>(new Uri[] { new Uri(CouchbaseConfigHelper.Instance.Server) });
	
        		config.BucketConfigs.Add(
            		CouchbaseConfigHelper.Instance.Bucket,
                new BucketConfiguration
                {
                    BucketName = CouchbaseConfigHelper.Instance.Bucket,
                    Username = CouchbaseConfigHelper.Instance.User,
                    Password = CouchbaseConfigHelper.Instance.Password
                });
	
        		config.BucketConfigs.Add(
            	"default",
            	new BucketConfiguration
            	{
                	BucketName = "default",
                	Username = CouchbaseConfigHelper.Instance.User,
                	Password = CouchbaseConfigHelper.Instance.Password
            	});
	
        		ClusterHelper.Initialize(config);
    		}
	
	       public static void Close()
	       {
	           ClusterHelper.Close();
	       }
		}
	} 
	```
			  
	The class `CouchbaseConfig` references a class called `CouchbaseConfigHelper` that does not yet exist. The purpose of `CouchbaseConfigHelper` class is to wrap calls to read the `AppSettings` section of `web.config` file. 
	
	The `appSettings` section is often used to store configuration settings for an application in .NET. In this tutorial we will follow this design practice from Microsoft.
	
3. In the project root create a new code file called: `CouchbaseConfigHelper.cs`. 
	Right click the project, in the menu select -> 'Add'-> 'Class'.
	
	![Add new file 3](content/images/Screen Shot 2015-11-18 at 09.40.22.png)

	![Add new file 3](content/images/Screen Shot 2015-11-18 at 09.40.50.png)
		
4. Replace the content of `CouchbaseConfigHelper.cs` with the following code snippet:

	```C#
	using System;
	using System.Collections.Generic;
	using System.Configuration;
	using System.Linq;
	using System.Web;
	
	namespace try_cb_dotnet
	{
	    public class CouchbaseConfigHelper
	    {
	        public CouchbaseConfigHelper()
	        {
	        }
	
	        private static CouchbaseConfigHelper instance = null;
	        public static CouchbaseConfigHelper Instance
	        {
	            get { if (instance == null) { instance = new CouchbaseConfigHelper(); } return instance; }
	        }
	
	        public string Bucket
	        {
	            get
	            {
	                return ConfigurationManager.AppSettings["couchbaseBucketName"];
	            }
	        }
	
	        public string Server
	        {
	            get
	            {
	                return ConfigurationManager.AppSettings["couchbaseServer"];
	            }
	        }
	
	        public string Password
	        {
	            get
	            {
	                return ConfigurationManager.AppSettings["couchbasePassword"];
	            }
	        }
	
	        public string User
	        {
	            get
	            {
	                return ConfigurationManager.AppSettings["couchbaseUser"];
	            }
	        }
	    }
	}
	```	
		
	Reading the code in 'CouchbaseConfigHelper' reveals that it's referencing a bunch of application setting keys in `web.config`, that we have still to create.	
	
5. Open `web.config` and add the missing application setting keys in the `appSettings` section as shown in the following snippet:	

	```XML	
	<configuration>
		<configSections>
    		...
	  	</configSections>
	  	<connectionStrings>
	    	...
	  	</connectionStrings>
	  	<appSettings>
		    <add key="webpages:Version" value="3.0.0.0" />
		    <add key="webpages:Enabled" value="false" />
		    <add key="ClientValidationEnabled" value="true" />
		    <add key="UnobtrusiveJavaScriptEnabled" value="true" />
		    <!-- COUCHBASE TRAVEL SAMPLE SETTINGS -->
		    <add key="couchbaseBucketName" value="travel-sample" />
		    <add key="couchbaseServer" value="http://localhost:8091" />
		    <add key="couchbasePassword" value="" />
		    <add key="couchbaseUser" value="" />
		    <!--END -->
	  	</appSettings>
	  	...
	</configuration>
	```

	You can change the settings in `web.config` to reflect your actual Couchbase Server setup. 
	
	This includes adding username and password if appropriate and correct the cluster URL if needed. 
	
	The current configuration assumes that Couchbase Server is installed on localhost and there's no password protection on the bucket.

6. We now have all configurations in place that we need to initialise the Couchbase .NET Client.
	Open the file `Global.asax.cs` in project root. 
	
7. Update the method `Application_Start()` with a call to `CouchbaseConfig.Initialize()` 
	This will initialise the Couchbasebase Client based on the settings from 'web.config'.

	```C#
	protected void Application_Start()
	{
        // Initialize Couchbase & ClusterHelper
        CouchbaseConfig.Initialize();
	
        AreaRegistration.RegisterAllAreas();
        GlobalConfiguration.Configure(WebApiConfig.Register);
        FilterConfig.RegisterGlobalFilters(GlobalFilters.Filters);
        RouteConfig.RegisterRoutes(RouteTable.Routes);
        BundleConfig.RegisterBundles(BundleTable.Bundles);
    }
	```
    
	The only thing missing now is disposing of resources when the application stops. This includes closing the connection to the Couchbase Cluster and releasing memory.
	
8. Update the `Application_End()` method to call the `Close()` method on `CouchbaseConfig`
	
	```C#
	protected void Application_End()
	{
		CouchbaseConfig.Close();
	}
	```
	
	You're ready to start using the Couchbase .NET Client. Happy coding!

###Step 2.1 

**Where:** `AirportController.cs` -> **method:** `FindAll(string search, string token)`

**Goals:** Return live data from the travel-sample data bucket in Couchbase Server and learn more about how to use Couchbase with .NET

**Relevant Documentation Topics:** 

* [Linq2Couchbase - GitHub](https://github.com/couchbaselabs/Linq2Couchbase)
* [Hello World - Couchbase .NET](http://developer.couchbase.com/documentation/server/4.0/sdks/dotnet-2.2/hello-couchbase.html)

**Task:**
In the current implementation the `FindAll(string search, string token)` method returns static data. 

Update the method to return data from the `travel-sample` bucket using the Couchbase .NET Client and N1QL.

The intention of `FindAll(string search, string token)` is to return n airport name based on the `search` string passed to the method. The `search` string can be in three different formats:

1. Three letter acronym: eg `LAX` 
2. Four letter acronym: eg `KLAX`
3. Free-text search with a partial all complet airport name: eg `Los Angeles Interna`'

In all three cases above the airport name returned should be `Los Angeles International Airport`.

Implement the method to return a list of airport names that match the `search` string
and return the result list in a JSON format like:

```JSON
[{"airportname":"San Francisco Intl"}]
```

The result list is used by the UI to show a drop-down of matching airport names the user can select from.

>Hint:
>
>Use N1QL to query the `travel-sample` bucket in Couchbase Server and return all matching airport names. 

**Solution:**

```C#
[HttpGet]
[ActionName("findAll")]
public object FindAll(string search, string token)
{
    if (search.Length == 3)
    {
        // LAX
        var query = 
            new QueryRequest("SELECT airportname FROM `" + CouchbaseConfigHelper.Instance.Bucket + "` WHERE faa=$1")
            .AddPositionalParameter(search.ToUpper());

        return ClusterHelper
            .GetBucket("travel-sample")
            .Query<dynamic>(query)
            .Rows;
    }
    else if (search.Length == 4)
    {
        // KLAX
        var query =
            new QueryRequest("SELECT airportname FROM `" + CouchbaseConfigHelper.Instance.Bucket + "` WHERE icao = '$1'")
            .AddPositionalParameter(search.ToUpper());

        return ClusterHelper
            .GetBucket(CouchbaseConfigHelper.Instance.Bucket)
            .Query<dynamic>(query)
            .Rows;
    }
    else
    {
        // Los Angeles
        var query =
            new QueryRequest("SELECT airportname FROM `" + CouchbaseConfigHelper.Instance.Bucket + "` WHERE airportname LIKE $1")
            .AddPositionalParameter("%" + search + "%");

        return ClusterHelper
            .GetBucket(CouchbaseConfigHelper.Instance.Bucket)
            .Query<dynamic>(query)
            .Rows;
    }
}    
```

Test the application with various airport names and abbreviations, you can start with: `SFO`, `KLAX`, `Los Angeles` etc.

###Step 2.2 

**Where:** `FlightPathController.cs` -> **method:** `FindAll(string from, DateTime leave, string to, string token)`

**Goals:** Return live data from the travel-sample data bucket in Couchbase Server and learn more about how to use Couchbase with .NET

**Relevant Documentation Topics:** 

* [Linq2Couchbase - GitHub](https://github.com/couchbaselabs/Linq2Couchbase)
* [Hello World - Couchbase .NET](http://developer.couchbase.com/documentation/server/4.0/sdks/dotnet-2.2/hello-couchbase.html)

**Task:**
In the current implementation `FindAll(string from, DateTime leave, string to, string token)` returns static data, using the `travel-sample` bucket, Couchbase .NET Client and N1QL we will update the method to return actual data from Couchbase Server. 

The intention of `FindAll(string from, DateTime leave, string to, string token)` is to search out and find actual route data. Trip data is created by joining `airport` documents with `routes`.

Implement the method to return all trips that match the selected source `from` and destination `to` airport name for the given date interval `leave`.

>Hint: 
>
>Use N1QL to query the `travel-sample` bucket in Couchbase Server to find matching `trips`.
>
>The `travel-sample` does not contain any `trip` documents therefore you will need to use `join` to build a result-set to represent `trips`.

This is a two step process, where each step will perform its own N1QL query.

1. This API method is called with the `from` and `to` values which represent the full airport names.

	The `join` that we will create in step 2, needs the FAA (three letter abbreviation for the airport name)

	Therefore the first step is to make a N1QL query to convert the airport name to a FAA, three letter value for both from and to airport.

2. This part is a bit tricky as it uses one of the more advanced features in N1QL, the `JOIN` and `UNNEST` statement.
	Below you can find a template version of the query.
	
    This will allow you to understand the query in its full detail and examin the parameters needed for this query.

```SQL
SELECT r.id, a.name, s.flight, s.utc, r.sourceairport, r.destinationairport, r.equipment FROM 
    `travel-sample` r 
    UNNEST r.schedule s 
    JOIN `travel-sample` a 
    ON KEYS r.airlineid 
    WHERE r.sourceairport='LAX' 
    AND r.destinationairport='SFO' 
    AND s.day=1
    ORDER BY a.name      
```
    
**Solution:**

```C#
[HttpGet]
[ActionName("findAll")]
public object FindAll(string from, DateTime leave, string to, string token)
{
    string queryFrom = null;
    string queryTo = null;
    var queryLeave = (int)leave.DayOfWeek;

    // raw query
    var query1 =
           new QueryRequest(
               "SELECT faa as fromAirport, geo FROM `" + CouchbaseConfigHelper.Instance.Bucket + "` " +
               "WHERE airportname = $from " +
               "UNION SELECT faa as toAirport, geo FROM `" + CouchbaseConfigHelper.Instance.Bucket + "` " +
               "WHERE airportname = $to")
           .AddNamedParameter("from", from)
           .AddNamedParameter("to", to);

    var partialResult1 = ClusterHelper
            .GetBucket(CouchbaseConfigHelper.Instance.Bucket)
            .Query<dynamic>(query1);

    if (partialResult1.Rows.Any())
    {
        foreach (dynamic row in partialResult1.Rows)
        {
            if (row.fromAirport != null) queryFrom = row.fromAirport;
            if (row.toAirport != null) queryTo = row.toAirport;
        }
    }

    // raw query
    var query2 =
           new QueryRequest(
               "SELECT r.id, a.name, s.flight, s.utc, r.sourceairport, r.destinationairport, r.equipment FROM " +
               "`" + CouchbaseConfigHelper.Instance.Bucket + "` r " +
               "UNNEST r.schedule s JOIN " +
               "`" + CouchbaseConfigHelper.Instance.Bucket + "` " +
               "a ON KEYS r.airlineid WHERE r.sourceairport=$from " +
               "AND r.destinationairport=$to " +
               "AND s.day=$leave " +
               "ORDER BY a.name")
           .AddNamedParameter("from", queryFrom)
           .AddNamedParameter("to", queryTo)
           .AddNamedParameter("leave", queryLeave);

    return ClusterHelper
            .GetBucket(CouchbaseConfigHelper.Instance.Bucket)
            .Query<dynamic>(query2)
            .Rows;
	}
}    
```
    
###Step 2.3

**Where:** `UserController.cs` -> **method:** `Flights(string token)`

**Goals:** Return live data about the user's travel bookings stored in Couchbase Server and learn more about how to use Couchbase with .NET

**Relevant Documentation Topics:** 

* [Linq2Couchbase - GitHub](https://github.com/couchbaselabs/Linq2Couchbase)
* [Hello World - Couchbase .NET](http://developer.couchbase.com/documentation/server/4.0/sdks/dotnet-2.2/hello-couchbase.html)

**Task:**
The intention of `Flights(string token)` is to return a list of all travel bookings stored by the user.

In the current implementation `Flights(string token)` returns static data.

Using the `travel-sample` bucket, Couchbase .NET Client and N1QL we will update the Web API method to return actual data stored about the user's bookings. 
All data is stored in Couchbase Server in the `travel-sample` bucket.

The `JWT token` id is used to look-up the user and find all bookings for that logged in user.

Response should be in JSON.

Implement the method to return all bookings for the logged-in user.

1. Use the token to look-up the "bookings" document for the user and return all bookings.
	The document key should be in a format like
	`bookings::token`
    Where `token` is the JWT Token id.
    
    The great thing with using Couchbase, as it is a JSON document store, is that we don't need to convert the value,
    we can just return the actual document retrieved from Couchbase Server.
 
>Hint:
> 
> Use `ClusterHelper` and `Get<dynamic>(...)` to retrive documents from the bucket and return the value. 
    
**Solution:**

```C#
[HttpGet]
[ActionName("flights")]
public object Flights(string token)
{
    return ClusterHelper
            .GetBucket(CouchbaseConfigHelper.Instance.Bucket)
            .Get<dynamic>("bookings::" + token)
            .Value;
}    
```

###Step 2.4

**Where:** `UserController.cs` -> **method:** `BookFlights([FromBody] dynamic request)`

**Goals:** Persist data about the user's travel bookings, persist them in Couchbase Server and learn more about how to use Couchbase with .NET

**Relevant Documentation Topics:** 

* [Linq2Couchbase - GitHub](https://github.com/couchbaselabs/Linq2Couchbase)
* [Hello World - Couchbase .NET](http://developer.couchbase.com/documentation/server/4.0/sdks/dotnet-2.2/hello-couchbase.html)

**Task:**
The intention of `BookFlights([FromBody] dynamic request)` is to store information about a users actual travel bookings.

In the current implementation `BookFlights([FromBody] dynamic request)` nothing is actually stored, we just return a constant that indicates how many line items where stored about a users bookings.
 
Using the `travel-sample` bucket, Couchbase .NET Client and N1QL we will update the method to store actual data about the users bookings. 

The JWT token is used as a key to the users bookings document, see step 2.3.

Responses should be in a JSON format like this:

```JSON
{"added":3}
```

Implement the method to return the the number of successful bookings that where persisted in Couchbase Server for the user. Remember to use the JWT token in the key for the document, like this:

`bookings::{token}`


1. First we need to get an understanding of the `dynamic` request value.
	
	Add a breakpoint inside the method and use Visual Studio's built in `Immediate Window` to investigate the request value.
	
    `request` is a list of dynamic `JToken` values. It's therefore possible to loop over the collection and select the the relevant values and store them directly in Couchbase.
2. Create a foreach loop to iterate over the request collection and store all bookings to Couchbase.
	
	The bookings should be stored in a document with the compound key
	`bookings::{token}`
	The `token` is available in the request parameter under the property, request.token
3. Update the return value to reflect the actual number of stored bookings.    

>Hint:
>
>Use `ClusterHelper` to `Upsert(...)` the document in the bucket. 
    
**Solution:**

```C#    
[HttpPost]
[ActionName("flights")]
public object BookFlights([FromBody] dynamic request)
{
    List<FlightModel> flights = new List<FlightModel>();

    foreach (var flight in request.flights)
    {
        flights.Add(new FlightModel
        {
            name = flight._data.name,
            bookedon = DateTime.Now.ToString(),
            date = flight._data.date,
            destinationairport = flight._data.destinationairport,
            sourceairport = flight._data.sourceairport
        });
    }

     ClusterHelper
        .GetBucket(CouchbaseConfigHelper.Instance.Bucket)
        .Upsert("bookings::" + request.token, flights);

    return new { added = flights.Count };
}
```

###Step 2 - Summery
In step 2 we learned how to bootstrap the .NET Couchbase Client and query data with N1QL using raw string queries. 

We have yet to learn how to use LINQ 2 Couchbase, we will come back to that in Step 4.

##Step 3 - Login Credentials and Authentications 
In this step we will implement the login page to use JWT Tokens for new and exciting users.

Before we can continue with the tutorial we first need to add a reference to the relevant NuGet packages to handle JWT Tokens. 

In short, JSON Web Tokens are an open, industry standard [RFC 7519](https://tools.ietf.org/html/rfc7519) method for representing claims securely between two parties.

In practice a JWT Token is a pice of JSON that can be signed and/or encrypted in a well defined and commonly understod way. Using a JWT Token exchange allows a receiver to verify the submitter and a secure way to share a information in the Token.    

This secure exchange implies the use of a key (shared or public/private). In our case we will use a shared key for encryption and signing. 

The JWT Token Secret is `UNSECURE_SECRET_TOKEN`. It is recommended that you chose something else for production a GUID, Cryptographic hash etc. But for our tutorial this will due. 

To maintain the design principles from Microsoft we will store the token in `Web.Config`, making it easy to change at a later point. 

###Step 3.0 - Add a reference to the JWT Token library

**Where:** `Solution` (this is a solution wide update)

**Goals:** Add a reference to: [JWT Token .NET](https://www.nuget.org/packages/JWT). 

Working with JWT Tokens is well defined and we could chose to implement our own library to work with JWT Tokens, but NuGet comes to the rescue.
The [JWT Token .NET](https://www.nuget.org/packages/JWT) that we will use seems to be the simplest and to the point library for our needs in this tutorial.
There are other libraries on NuGet, feel free to search for others.

**Relevant Documentation Topics:** 

* [JWT Token .NET](https://www.nuget.org/packages/JWT)

###3.0 - Referencing the JWT Token library
For more information about the package and developers behind it, visit
[NuGet Gallery](https://www.nuget.org/packages). 

NuGet is a centralised repository for package authors and consumers, and it also defines a suite of tools for authoring, publishing and consuming packages from various vendors and authors.

In step 2.0, we used Visual Studios build in graphical view to add the Couchbase NuGet Packages, this time we will use the `Package Manager Console` a command line tool build into Visual Studio. 

This is just to show an alternativ, the result is the same.

Using Visual Studio 2015 or later, follow these steps to get the JWT Token library added:

1. From the IDE top menu, select -> View -> Other Windows -> `Package Manager Console`.

	![Package Manager Console](content/images/Screen Shot 2015-11-19 at 11.15.27.png)

2. The `Package Manager Console` opens in the bottom part of the IDE, like this:

	![NuGet view](content/images/Screen Shot 2015-11-19 at 11.22.50.png)
	
	Please note the yellow message box "Some NuGet packages are missing from this....". This message will most likely not show up in your solution as all packages are already installed. 
	
	This is just included to show that all features (and more actually) are available in console mode.

3. To install a package we use the `Install-Package` command. Copy/paste the below command to the `Package Manager Console` window and press Enter.
	`Install-Package JWT`

	![NuGet install JWT](content/images/Screen Shot 2015-11-19 at 11.28.40.png)

	That’s it! NuGet has pulled in all required dependencies and reference required dependencies for the JWT Token library. 

###Step 3.1

**Where:** `UserController.cs` -> **method:** `Login(string password, string user)`

**Goals:** 
Update the Web Api method to return a valid JWT token and store it in Couchbase Server for the individual users.

**Relevant Documentation Topics:** 

* [JWT Token .NET](https://www.nuget.org/packages/JWT)

**Task:**    

This is a Web API call, a method that is called from the static html (index.html).
The JavaScript in the static html page expects this `Login` Web API call to return a
"success" status code containing a valid JWT token.
 
The JWT token is used as as reference to store data about the user bookings and login credentials.

Response should be in a JSON format.

Implement the method to return a "success" allowing the user to login if credentials are valid.

1. 	Use ClusterHelper to look-up the user document and validate the login. 
	If the login is valid then return 
	"Success" : token
    If not, return 
    "Success" : false
    
>Hint:
>
>The user document is stored under the key: "profile::{user}" in the bucket.

**Solution:**

```C#
[HttpGet]
[ActionName("Login")]
public object Login(string password, string user)
{
    try
    {
        var result = ClusterHelper
            .GetBucket("default")
            .Get<dynamic>("profile::" + user);

        if (result.Success && result.Status == Couchbase.IO.ResponseStatus.Success && result.Exception == null && result.Value != null)
        {
            var jsonDecodedTokenString =
                JsonWebToken
                .Decode(result.Value, CouchbaseConfigHelper.Instance.JWTTokenSecret, false);

            var jwtToken = JsonConvert.DeserializeAnonymousType(jsonDecodedTokenString, new { user = "", iat = "" });

            if (jwtToken.iat == password)
            {
                return new { success = result.Value };
            }
        }
    }
    catch (Exception)
    {
        // Silence the Exception
    }

    return new { success = false };
}
```

###Step 3.2

**Where:** `UserController.cs` -> **method:** `CreateLogin([FromBody] UserModel user)`

**Goals:** 
Update the Web Api method to return JWToken stored in Couchbase Server for then individual users.

**Relevant Documentation Topics:** 

* [JWT Token .NET](https://www.nuget.org/packages/JWT)

**Task:**    

This is a Web API call, a method that is called from the static html (index.html).
The JS in the static html expects this "Login" web api call to return a
"success" status code containing a JWT token. 

The JWT token is used to reference and store data about the user's trips/booking and login credentials.
Response should be in a JSON format.

Implement the method to create and return a valid JWT Token given the username and password.
Be sure to check if a user already exists and fail in that case by returning "success" : false

1. Check if user document already exists and fail if it does.
2. Visit the [JWT Token library readme](https://github.com/jwt-dotnet/jwt)
3. Read the documentation and learn how to use the library to create and validate a JWT Token.
4. Use the secret key for encryption and hashing:
    "UNSECURE_SECRET_TOKEN"
5. Store the secret key in `Web.Config`.
6. Store the generated JWT token under the user document key:
    "profile::user" in the bucket.
7. Update the method to return the actual JWT token.

>Hint:
>
> Use `ClusterHelper` to store the document. 


**Solution:**

**UserController.cs**

```C#
[HttpPost]
[ActionName("Login")]
public object CreateLogin([FromBody] UserModel user)
{
    try
    {
        if (ClusterHelper.GetBucket("default").Exists("profile::" + user.User))
        {
            throw new Exception("User already Exists!");
        }

        string jsonToken =
            JsonWebToken
            .Encode(
                new { user = user.User, iat = user.Password },
                CouchbaseConfigHelper.Instance.JWTTokenSecret,
                JwtHashAlgorithm.HS512);

        var result = ClusterHelper
            .GetBucket("default")
            .Upsert<dynamic>("profile::" + user.User, jsonToken);

        if (!result.Success || result.Exception != null)
        {
            throw new Exception("could not save user to Couchbase");
        }

        return new { success = jsonToken };
    }
    catch (Exception)
    {
        // Silence the Exception
    }

    return new { success = false };
}
```
    
**CouchbaseConfigHelper.cs**

```C#
.
.
.
public string JWTTokenSecret
{
    get
    {
        return ConfigurationManager.AppSettings["JWTTokenSecret"];
    }
}
.
.
.
```

**Web.Config**

```XML
<appSettings>
	...
	<!-- COUCHBASE TRAVEL SAMPLE SETTINGS -->
	...
	<add key="JWTTokenSecret" value="UNSECURE_SECRET_TOKEN" />
	<!--END -->
</appSettings>
```

###Step 3 - Summery
In part 3 we added JWT Tokens and login credentials to the web site, allowing users to create profiles and store/retrieve their bookings.

##Step 4 - Using LINQ with N1QL
In this step we will update one of the raw N1QL queries to use the LINQ extensions for N1QL. 

LINQ is an abbreviation for Language Integrated Query. LINQ allows you as a developer to take query into your .NET code and describe your search and set conditions.

If you want to learn more about LINQ and it's impressive capabilities, then please visit the official Microsoft MSDN site, [here](https://msdn.microsoft.com/en-us/library/bb397926.aspx).  

###Step 4.1

**Where:** `FlightPathController.cs` -> **method:** `FindAll(string from, DateTime leave, string to, string token)`

**Goals:** 
Update one Web API method to use LINQ with N1QL to query the data, instead of the raw queries used so far. 

**Relevant Documentation Topics:** 

* [LINQ 2 Couchbase - blog post](http://blog.couchbase.com/2015/august/introducing-linq2couchbase-developer-preview-1-the-linq-provider-for-couchbase-n1ql)

**Task:**  
Implement the method to use the `Linq2Couchbase` extensions for LINQ. You can choose to use `Query syntax` or `Lambda syntax`.

Read more about [Lambda expressions](https://msdn.microsoft.com/en-us/library/bb397687.aspx) and [Query syntax](https://msdn.microsoft.com/en-us/library/bb397947.aspx). 

Before we can start using LINQ to query our documents, we need to create class that describe the content of the documents. This is one drawback using LINQ, but the gain is type-safety and code completion.

Using an online JSON -> C# class converter, makes it very easy to create the classes.
[JSON2CSHARP](http://json2csharp.com/)

LINQ requires this model to base it's query upon (needed for reflection). Therefore you will need to create a PoCo for every document you will query with LINQ and or return as a result from a query.

1. Create a PoCo Class for the LINQ queries for the document type `Airport`.
	
	Add a new Class to the project and name it `Airport.cs`
	Replace the content of the file with the following snippet:
	
	```C#
    [EntityTypeFilter("airport")]
    public class Airport
    {
        [JsonProperty("airportname")]
        public string Airportname { get; set; }
    
        [JsonProperty("city")]
        public string City { get; set; }
    
        [JsonProperty("country")]
        public string Country { get; set; }
    
        [JsonProperty("faa")]
        public string Faa { get; set; }
    
        [JsonProperty("geo")]
        public Geo Geo { get; set; }
    
        [JsonProperty("icao")]
        public string Icao { get; set; }
    
        [JsonProperty("id")]
        public string Id { get; set; }
    
        [JsonProperty("type")]
        public string Type { get; set; }
    
        [JsonProperty("tz")]
        public string Tz { get; set; }
    }
	```
2. Create a PoCo Class for the LINQ queries for the document type `Geo`.
	`Geo`is a complex type used by `Airport`.
	
	Add a new Class to the project and name it `Geo.cs`
	Replace the content of the file with the following snippet:	
	```C#
	[EntityTypeFilter("geo")]
	public class Geo
	{
	    [JsonProperty("alt")]
	    public double Alt { get; set; }
	///
	    [JsonProperty("lat")]
	    public double Lat { get; set; }
	///
	    [JsonProperty("lon")]
	    public double Lon { get; set; }
	}
	```
	
3. Create the query using Lambda or Query syntax.

	Read the [LINQ 2 Couchbase - blog post](http://blog.couchbase.com/2015/august/introducing-linq2couchbase-developer-preview-1-the-linq-provider-for-couchbase-n1ql) to get more information about how to query with the LINQ provider.

**Solution:**    

```C#
public class FlightPathController : ApiController
{
[HttpGet]
[ActionName("findAll")]
public object FindAll(string from, DateTime leave, string to, string token)
{
    // query syntax
    var airlinesQuerySyntax
        = (from fromAirport in ClusterHelper.GetBucket(CouchbaseConfigHelper.Instance.Bucket).Queryable<Airport>()
           where fromAirport.Airportname == @from
           select new { fromAirport = fromAirport.Faa, geo = fromAirport.Geo })
                    .ToList() // need to execute the first part of the select before call to Union
                   .Union<dynamic>(
                            from toAirport in ClusterHelper.GetBucket(CouchbaseConfigHelper.Instance.Bucket).Queryable<Airport>()
                            where toAirport.Airportname == to
                            select new { toAirport = toAirport.Faa, geo = toAirport.Geo });

    // lambda syntax
    var airlinesLambdaSyntaxt
        = ClusterHelper.GetBucket(CouchbaseConfigHelper.Instance.Bucket).Queryable<Airport>()
        .Where(airline => airline.Airportname == @from)
        .Select(airline => new { fromAirport = airline.Faa, geo = airline.Geo })
        .ToList() // need to execute the first part of the select before call to Union
        .Union<dynamic>(
                ClusterHelper.GetBucket(CouchbaseConfigHelper.Instance.Bucket).Queryable<Airport>()
                .Where(airline => airline.Airportname == to)
                .Select(airline => new { toAirport = airline.Faa, geo = airline.Geo })
                );

    //var airlinesResult = airlinesLambdaSyntaxt.ToList();
    var airlinesResult = airlinesQuerySyntax.ToList();

    string queryFrom = null;
    string queryTo = null;
    var queryLeave = (int)leave.DayOfWeek;

    foreach (var row in airlinesResult)
    {
        try
        {
            if (row.fromAirport != null) queryFrom = row.fromAirport;
        }
        catch (Exception)
        {
            // silence the exception as this is known to throw one time,
            // for the row that does not contain toAirport. 
            // There is no easy way to test for the missing attribute on the 
            // dynamic type.
        }

        try
        {
            if (row.toAirport != null) queryTo = row.toAirport;
        }
        catch (Exception)
        {
            // silence the exception as this is known to throw one time,
            // for the row that does not contain fromAirport. 
            // There is no easy way to test for the missing attribute on the 
            // dynamic type.
        }
    }

    // raw query
    var query2 =
           new QueryRequest(
               "SELECT r.id, a.name, s.flight, s.utc, r.sourceairport, r.destinationairport, r.equipment FROM " +
               "`" + CouchbaseConfigHelper.Instance.Bucket + "` r " +
               "UNNEST r.schedule s JOIN " +
               "`" + CouchbaseConfigHelper.Instance.Bucket + "` " +
               "a ON KEYS r.airlineid WHERE r.sourceairport=$from " +
               "AND r.destinationairport=$to " +
               "AND s.day=$leave " +
               "ORDER BY a.name")
           .AddNamedParameter("from", queryFrom)
           .AddNamedParameter("to", queryTo)
           .AddNamedParameter("leave", queryLeave);

    return ClusterHelper
            .GetBucket(CouchbaseConfigHelper.Instance.Bucket)
            .Query<dynamic>(query2)
            .Rows;
}
```

###Step 4 - Summary
In part 4 you learned how to use LINQ with N1QL. 

The benefits of using LINQ with N1QL is code completion, type safety and compile time checks of you queries. On the other hand there is an extra step required when creating the PoCo classes in converting the JSON structures to C# class's. 

It's entirely up to you to decide what approach fits best to your temper and or projects. 
 
##Step 5 - Done
This is the Travel sample app in it's entirety, nothing needs to be updated or changed. 

You can find the final version here:

[Step 5/branch 5](https://github.com/couchbaselabs/try-cb-dotnet/tree/tutorial-part-5)

Use this branch as a reference when creating the previous steps or as a reference app. 

#Comments and Feedback
If you have any comments to the source, tutorial or any content related to this tutorial and/or source please open an ticket in the issues section on GitHub, [here](https://github.com/couchbaselabs/try-cb-dotnet/issues)

Thanks for trying Couchbase with .NET!

