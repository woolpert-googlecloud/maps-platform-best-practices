---
title: Place Details Field Filtering
---

# Place Details Field Filtering

Pricing for Places API requests is tiered and based on which categories of fields are returned. Google differentiates between **Basic**, **Contact**, and **Atmosphere** fields. Basic data is included in the cost of each Places API service but requesting Contact and/or Atmosphere fields costs extra.

> 📚 For more on how Places API requests are billed, see our guide: [Understanding Places API Billing](places-api-billing)

In June 2018, Google introduced a new `fields` parameter to Places API, so developers can control what is returned, and only pay for data they need. Because this is an additional parameter, you will have to change your code to filter your requests and reduce your cost.

> ⚠️ For backward compatibility, Places API returns **all fields** by default.

Your existing applications will be billed for all three categories of data if you don't change your code to add field filtering. This affects all users of the places library for the *Google Maps JavaScript API* (including *Place Autocomplete*), the mobile *Places SDKs for Android & iOS*, and all *web service* requests.

This guide will help you determine which fields you need and how to add field filtering to your code.


## Which Fields Do You Need?

Here is a breakdown of the fields included in each category. Evaluate which of these are being used in your application, and note which category they fall into.

| SKU | Fields |
| --- | --- |
| **Basic** | `address_component`, `adr_address`, `alt_id`, `formatted_address`, `geometry` (location), `icon`, `id`, `name`, `permanently_closed`, `photo`, `place_id`, `plus_code`, `scope`, `type`, `url`, `utc_offset`, `vicinity` |
| **Contact** | `formatted_phone_number`, `international_phone_number`, `opening_hours`, `website` |
| **Atmosphere** | `price_level`, `rating`, `review`, `user_ratings_total` |

See the [Place Details results documentation](https://developers.google.com/places/web-service/details#PlaceDetailsResults) for detailed descriptions of each field.

## Filter Your Places API Requests

### Web Service Requests
When making **Place Details** requests through the web service, add a `field=` parameter to the URL, followed by a comma-separated list of the fields you want returned.

***Example** - Return only the name, address, and location for a place (**Basic**):*
```
https://maps.googleapis.com/maps/api/place/details/json?placeid=ChIJlQ-ho9OEQIgRLQeLM-VMRMc&fields=name,formatted_address,address_component,geometry&key=YOUR_API_KEY
```

***Example** - Return only the name and hours of operation for a place (**Basic + Contact**):*
```
https://maps.googleapis.com/maps/api/place/details/json?placeid=ChIJlQ-ho9OEQIgRLQeLM-VMRMc&fields=name,opening_hours&key=YOUR_API_KEY
```

***Example** - Return only the name and rating for a place (**Basic + Atmosphere**):*
```
https://maps.googleapis.com/maps/api/place/details/json?placeid=ChIJlQ-ho9OEQIgRLQeLM-VMRMc&fields=name,rating&key=YOUR_API_KEY
```

> ⚠️ **Nearby** and **Text Searches** do not have field filtering capability, they will always generate Contact and Atmosphere charges.

### Google Maps JavaScript API

The Places library for the Google Maps JavaScript API has field filtering built in.

#### Place Autocomplete Widget

If you're using Google's built-in Place Autocomplete widget, set a filter by adding the `fields` property to the [options object](https://developers.google.com/maps/documentation/javascript/reference/places-widget#AutocompleteOptions) you pass to the `Autocomplete` constructor. The property is an `Array<string>`, containing the names of the fields you want returned.

```js
var autocomplete = new google.maps.places.Autocomplete(
  document.getElementById("autocomplete-input"),
  {
    fields: ["name", "address_component", "formatted_address", "geometry.location"]
  }
);
```

You can also change the filter at any time by calling the `setFields` function and passing an `Array<string>` of field names.
```js
autocomplete.setFields(["name", "formatted_address", "opening_hours"]);
```

> ⚠️ The Places `SearchBox` does not support field filtering, and will be billed for all three categories of data. If possible, use `Autocomplete` instead.

#### Places Service

If you are using a `PlacesService` instance for **Find Place** or **Place Details** queries (including `AutocompleteService` implementations), set a filter by adding the `fields` property to the [request object](https://developers.google.com/maps/documentation/javascript/reference/places-service#PlaceDetailsRequest) you pass to the following functions: `findPlaceFromPhoneNumber`, `findPlaceFromQuery`, `getDetails`.

> ⚠️ The `nearbySearch` and `textSearch` functions do not support field filtering, and will be billed for all three categories of data.

```js
var placesService = new google.maps.places.PlacesService(document.getElementById("places-attr-container"),
placesService.getDetails({
  placeId: "ChIJlQ-ho9OEQIgRLQeLM-VMRMc",
  fields: ["name", "address_component", "formatted_address", "geometry.location"]
}, callback);
```

### Places Mobile SDKs for Android/iOS
> ⚠️ On January 29, 2019 Google released new versions of the Places SDKs for both Android and iOS. These new SDKs are officially supported on Maps Platform and include the latest Places API functionality, such a s field filtering. The previous SDKs are now deprecated and developers have until **July 29, 2019** to upgrade their applications. See [this blog post](https://woolpert.com/resource/welcome-to-the-google-maps-platform-places-sdk/) for details.

#### Android
If you are using Google's built-in Autocomplete widget, configure the fields you want when you initialize the intent. 
```java
List<Place.Field> fields = Arrays.asList(Place.Field.NAME, Place.Field.ADDRESS_COMPONENTS, Place.Field.ADDRESS, Place.Field.LAT_LNG);

Intent intent = new Autocomplete.IntentBuilder(AutocompleteActivityMode.FULLSCREEN, fields).build(this);
```
Or, if you are retrieving Place Details programatically, include the fields you want in your `FetchPlaceRequest` or `FindCurrentPlaceRequest`.
```java
String placeId = "ChIJlQ-ho9OEQIgRLQeLM-VMRMc";
List<Place.Field> fields = Arrays.asList(Place.Field.NAME, Place.Field.ADDRESS_COMPONENTS, Place.Field.ADDRESS, Place.Field.LAT_LNG);

// Fetch Place by ID
FetchPlaceRequest detailsRequest = FetchPlaceRequest.builder(placeId, fields).build();
// Find User's Current Place
FindCurrentPlaceRequest currentPlaceRequest = FindCurrentPlaceRequest.builder(fields).build();
```

#### iOS
If you are using Google's built-in Autocomplete widget, configure the fields you want when you initialize the controller.
```swift
let autocompleteController = GMSAutocompleteViewController()
autocompleteController.delegate = self

let fields: GMSPlaceField = GMSPlaceField(rawValue: UInt(GMSPlaceField.name.rawValue) |  UInt(GMSPlaceField.addressComponents.rawValue) |  UInt(GMSPlaceField.formattedAddress.rawValue) |  UInt(GMSPlaceField.coordinate.rawValue))!
autocompleteController.placeFields = fields
```
Or, if you are retrieving Place Details programatically, include the fields you want in your `fetchPlace()` or `findPlaceLikelihoodsFromCurrentLocation()` request.
```swift
let placeID = "ChIJlQ-ho9OEQIgRLQeLM-VMRMc"
let fields: GMSPlaceField = GMSPlaceField(rawValue: UInt(GMSPlaceField.name.rawValue) |  UInt(GMSPlaceField.addressComponents.rawValue) |  UInt(GMSPlaceField.formattedAddress.rawValue) |  UInt(GMSPlaceField.coordinate.rawValue))!

// Fetch Place by ID
placesClient?.fetchPlace(fromPlaceID: placeId, placeFields: fields, sessionToken: nil, callback: {...})
// Find User's Current Place
placesClient?.findPlaceLikelihoodsFromCurrentLocation(withPlaceFields: fields, callback: {...})
```


## References
[Maps Platform "Guide for Existing Users": Places API Product Updates](https://cloud.google.com/maps-platform/user-guide/product-changes/#places)

[Place Details Fields Documentation](https://developers.google.com/places/web-service/details#fields)