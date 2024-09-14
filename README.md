# Fast-Weigh 3rd Party Loadout

This is the API contract for the Fast-Weigh Loadout third-party interface. This document outlines the expected behavior of the API endpoints that the Fast-Weigh Loadout UI will interact with.

This spec is designed as a RESTful API, with endpoints that accept and return JSON.

> [!NOTE]
> All payloads below are shown with type annotations. These annotations should not be included in the actual request or response payloads.

## Integration Setup

In Fast-Weigh, each silo must be assigned a **Base URL** and **Silo ID**. These are entered in the Fast-Weigh Server Settings. An optional API Key can be provided if authentication is required.

**Base URL**: The base url and/or port for the third-party API service (Ex: http://localhost:5500). Please note that port 5001 is reserved for the Fast-Weigh server instance.

**Silo ID**: The unique identifier for the silo. This is used to identify the silo in the API requests.

**API Key**: An optional API key that can be provided if the third-party API requires authentication. This key is passed in the `x-api-key` request header, and as a query parameter in the request URL with every request. Ex:

```http
GET /serverstatus?key=supersecretkey
x-api-key: supersecretkey
```

## Error Handling

Errors should be returned with a 400 or 500 level HTTP status code and a JSON object with a message property. The message will be displayed in the Fast-Weigh UI and should provide a human readable description of the error.

```json
{
	"message": "Error message here" // string
}
```

## API Implementation Notes

> [!IMPORTANT]
> The Fast-Weigh application can be run on any network IP and runs in the browser. Your API Implementation MUST allow `GET` and `POST` requests from any origin.

**Helpful Links**:

- [Learn about CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)
- [How to allow cross-origin requests in C# ASP.NET project](https://learn.microsoft.com/en-us/aspnet/core/security/cors?view=aspnetcore-8.0)

**Sample Middleware**:

```cs
var MyAllowSpecificOrigins = "_myAllowSpecificOrigins";

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddCors(options =>
{
  options.AddPolicy(MyAllowSpecificOrigins,
    policy =>
      {
          policy.WithOrigins("*")
            .AllowAnyHeader()
            .AllowAnyMethod();
      });
});

builder.Services.AddControllers();

var app = builder.Build();
app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();

app.UseCors(MyAllowSpecificOrigins);

app.UseAuthorization();

app.MapControllers();

app.Run();
```

## Required Endpoints

These endpoints are required for the integration to function properly.

### Server Status

This endpoint is used to check the status of the third-party API server. It should return a 200 OK response if the server is running and available. Any non-200 response will be considered an error and all loadout operations will be disabled.

**Request**:

```http
GET /serverstatus
```

**Response**:

```http
200 OK
```

### Silo Status

This endpoints returns the current status of the specified silo. It should return inventory information, as well as the loadout status.

**Request**:

```http
GET /silos
```

**Response**:

```json
[
  {
    "id": "silo1"              // string -- unique id for the silo
    "isActive": true,          // boolean
    "isSafeForLoadout": true,  // boolean -- this field should come from the 3rd party's own safety checks
    "isFilling: true,          // boolean -- optional: silo is currently being filled with inventory
    "currentInventory": 0.0,   // number
    "unitOfMeasure": "TONS"    // string -- ENUM: 'TONS' | 'LBS' | 'KGS' | 'TONNES' 
  }
]
```





