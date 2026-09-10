# Geo Coder (`base.geocoder`)

**Transport name:** `base.geocoder`  
**Storage name:** `base_geocoder`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `base_geolocalize`

Description: Geo Coder

## Operations (10)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_provider` | preparation rule | self | `base_geolocalize` | model |  |
| `geo_query_address` | operation | self, street, zip, city, state, country | `base_geolocalize` | model | Converts address fields into a valid string for querying geolocation APIs. :param street: street address :param zip: zip code :param city: city :param state: state :param country: country :return: formatted string |
| `geo_find` | operation | self, addr, **kw | `base_geolocalize` | model | Use a location provider API to convert an address string into a latitude, longitude tuple. Here we use Openstreetmap Nominatim by default. :param addr: Address string passed to API :return: (latitude, longitude) or None if not found |
| `_call_openstreetmap` | internal rule | self, addr, **kw | `base_geolocalize` | model | Use Openstreemap Nominatim service to retrieve location :return: (latitude, longitude) or None if not found |
| `_call_openstreetmap_reverse` | internal rule | self, lat, lon | `base_geolocalize` | model | Use Openstreemap Nominatim service to retrieve location from latitude and longitude :param lat: Latitude :param lon: Longitude :return: Address string or None if not found |
| `_call_googlemap` | internal rule | self, addr, **kw | `base_geolocalize` | model | Use google maps API. It won't work without a valid API key. :return: (latitude, longitude) or None if not found |
| `_geo_query_address_default` | internal rule | self, street, zip, city, state, country | `base_geolocalize` | model |  |
| `_geo_query_address_googlemap` | internal rule | self, street, zip, city, state, country | `base_geolocalize` | model |  |
| `_raise_query_error` | internal rule | self, error | `base_geolocalize` |  |  |
| `_get_localisation` | preparation rule | self, latitude, longitude | `base_geolocalize` |  |  |

## Validation and error messages (5)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `geo_find` | UserError | Provider %s is not implemented for geolocation service. | `base_geolocalize` |
| `_call_openstreetmap_reverse` | UserError | OpenStreetMap calls disabled in testing environment. | `base_geolocalize` |
| `_call_googlemap` | UserError | API key for GeoCoding (Places) required. Visit https://developers.google.com/maps/documentation/geocoding/get-api-key for more information. | `base_geolocalize` |
| `_call_googlemap` | UserError | error_msg | `base_geolocalize` |
| `_raise_query_error` | UserError | Error with geolocation server: %s | `base_geolocalize` |

Machine-readable definition: `../../../schemas/data/entities/base.geocoder.json`.
