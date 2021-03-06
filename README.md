# Swashbuckle.Examples
A simple library which adds the `[SwaggerRequestExample]`, `[SwaggerResponseExample]` attributes to [Swashbuckle](https://github.com/domaindrivendev/Swashbuckle). Can also add read the `[Description]` attributes off your Response objects, and can also add an input box for entering an Authorization header.

Blog articles: https://mattfrear.com/2016/01/25/generating-swagger-example-requests-with-swashbuckle/
and: https://mattfrear.com/2015/04/21/generating-swagger-example-responses-with-swashbuckle/

## Request example

Populate swagger's `definitions.YourObject.example` with whatever object you like.

This is great for manually testing and demoing your API as it will prepopulate the request with some useful data, so that 
when you click the example request in order to populate the form, instead of getting an autogenerated request like this:

![autogenerated request with crappy data](https://mattfrear.files.wordpress.com/2016/01/untitled.png?w=700)

You�ll get your desired example, like this:

![custom request with awesome data](https://mattfrear.files.wordpress.com/2016/01/capture2.jpg?w=700)

You can see the example output in the underlying swagger.json file, which you can get to by starting your solution and 
navigating to /swagger/docs/v1

![swagger.json](https://mattfrear.files.wordpress.com/2016/01/capture.jpg)

## Response example

Allows you to add custom data to the example response shown in Swagger. So instead of seeing the default boring data like so:

![response with crappy data](https://mattfrear.files.wordpress.com/2015/04/response-old.png)

You'll see some more realistic data (or whatever you want):

![response with awesome data](https://mattfrear.files.wordpress.com/2015/04/response-new.png?w=700&h=358)

## Documenting Response properties (new!)

Lets you add a comment-like description to properties on your response, e.g.
![descriptions](https://mattfrear.files.wordpress.com/2017/09/descriptions.jpg)

## Authorization header input box (new!)

Adds an input so that you can send an Authorization header to your API. Useful for API endpoints that have JWT token
authentication. e.g.

![authorization](https://mattfrear.files.wordpress.com/2017/09/authorization.jpg)

## Installation
Install the [NuGet package](https://www.nuget.org/packages/Swashbuckle.Examples/), then enable whichever filters 
you need when you enable Swagger:

```
GlobalConfiguration.Configuration 
    .EnableSwagger(c =>
        {
            c.SingleApiVersion("v1", "WebApi");

            // Enable Swagger examples
            c.OperationFilter<ExamplesOperationFilter>();

            // Enable swagger response descriptions
            c.OperationFilter<DescriptionOperationFilter>();
        })

```

## How to use - Request examples

Decorate your controller methods with the included SwaggerRequestExample attribute:

```
[SwaggerRequestExample(typeof(DeliveryOptionsSearchModel), typeof(DeliveryOptionsSearchModelExample))]
public async Task<IHttpActionResult> DeliveryOptionsForAddress(DeliveryOptionsSearchModel search)
```

Now implement it, in this case via a DeliveryOptionsSearchModelExample (which should implement IExamplesProvider), 
which will generate the example data. It should return the type you specified when you specified the `[SwaggerRequestExample]`.
	
```
public class DeliveryOptionsSearchModelExample : IExamplesProvider
{
    public object GetExamples()
    {
        return new DeliveryOptionsSearchModel
        {
            Lang = "en-GB",
            Currency = "GBP",
            Address = new AddressModel
            {
                Address1 = "1 Gwalior Road",
                Locality = "London",
                Country = "GB",
                PostalCode = "SW15 1NP"
            },
            Items = new[]
            {
                new ItemModel
                {
                    ItemId = "ABCD",
                    ItemType = ItemType.Product,
                    Price = 20,
                    Quantity = 1,
                    RestrictedCountries = new[] { "US" }
                }
            }
        };
    }
```

## How to use - Response examples

Decorate your methods with the new SwaggerResponseExample attribute:
```
[SwaggerResponse(HttpStatusCode.OK, Type=typeof(IEnumerable<Country>))]
[SwaggerResponseExample(HttpStatusCode.OK, typeof(CountryExamples))]
[SwaggerResponse(HttpStatusCode.BadRequest, Type = typeof(IEnumerable<ErrorResource>))]
public async Task<HttpResponseMessage> Get(string lang)
```

Now you�ll need to add an Examples class, which will implement IExamplesProvider to generate the example data

```	
public class CountryExamples : IExamplesProvider
{
    public object GetExamples()
    {
        return new List<Country>
        {
            new Country { Code = "AA", Name = "Test Country" },
            new Country { Code = "BB", Name = "And another" }
        };
    }
}
```
### Known issues
- Although you can add a response examples for each HTTP status code (200, 400, 404 etc), and they will appear in the
swagger.json, they will not display correctly. This is due to an bug in swagger-ui. [Issue 9](https://github.com/mattfrear/Swashbuckle.AspNetCore.Examples/issues/9)

## How to use - Document response properties
Define the SwaggerResponse, as usual:
```
[HttpPost]
[Route("api/values/person")]
[SwaggerResponse(200, typeof(PersonResponse), "Successfully found the person")]
public PersonResponse GetPerson([FromBody]PersonRequest personRequest)
{
```
Now add `System.ComponentModel.Description` attributes to your Response object:
```
public class PersonResponse
{
	public int Id { get; set; }

	[Description("The first name of the person")]
	public string FirstName { get; set; }

	public string LastName { get; set; }

	[Description("His age, in years")]
	public int Age { get; set; }
```

## How to use - Authorization input

Just enable the `AuthorizationInputOperationFilter` as described in the Installation section above. Note this this will add an
Authorization input to EVERY endpoint, regardless of if the endpoint is actually secured.

## Pascal case or Camel case?
The default is camelCase. If you want PascalCase you can pass in a `DefaultContractResolver` like so:
`[SwaggerResponseExample(200, typeof(PersonResponseExample), typeof(DefaultContractResolver))]`

## Render Enums as strings
By default `enum`s will output their integer values. If you want to output strings you can pass in a `StringEnumConverter` like so:
`[SwaggerResponseExample(200, typeof(PersonResponseExample), jsonConverter: typeof(StringEnumConverter))]`