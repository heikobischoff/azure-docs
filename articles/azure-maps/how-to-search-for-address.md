---
title: Search for a location using Azure Maps Search services
description: Learn about the Azure Maps Search service. See how to use this set of APIs for geocoding, reverse geocoding, fuzzy searches, and reverse cross street searches.
author: eriklindeman
ms.author: eriklind
ms.date: 10/28/2021
ms.topic: how-to
ms.service: azure-maps
services: azure-maps

---

# Search for a location using Azure Maps Search services

The [Search service] is a set of RESTful APIs designed to help developers search addresses, places, and business listings by name, category, and other geographic information. In addition to supporting traditional geocoding, services can also reverse geocode addresses and cross streets based on latitudes and longitudes. Latitude and longitude values returned by the search can be used as parameters in other Azure Maps services, such as [Route] and [Weather] services.

In this article, you'll learn how to:

* Request latitude and longitude coordinates for an address (geocode address location) by using [Search Address].
* Search for an address or Point of Interest (POI) using [Fuzzy Search].
* Use [Reverse Address Search] to translate coordinate location to street address.
* Translate coordinate location into a human understandable cross street using [Search Address Reverse Cross Street].  Most often, this is needed in tracking applications that receive a GPS feed from a device or asset, and wish to know where the coordinate is located.

## Prerequisites

1. An [Azure Maps account]
2. A [subscription key]

This tutorial uses the [Postman] application, but you may choose a different API development environment.

## Request latitude and longitude for an address (geocoding)

In this example, we'll use [Get Search Address] to convert an address into latitude and longitude coordinates. This process is also called *geocoding*. In addition to returning the coordinates, the response will also return detailed address properties such as street, postal code, municipality, and country/region information.

>[!TIP]
>If you have a set of addresses to geocode, you can use [Post Search Address Batch] to send a batch of queries in a single request.

1. In the Postman app, select **New** to create the request. In the **Create New** window, select **HTTP Request**. Enter a **Request name** for the request.

2. Select the **GET** HTTP method in the builder tab and enter the following URL. In this request, we're searching for a specific address: `400 Braod St, Seattle, WA 98109`. For this request, and other requests mentioned in this article, replace `{Your-Azure-Maps-Subscription-key}` with your Azure Maps subscription key.

    ```http
    https://atlas.microsoft.com/search/address/json?&subscription-key={Your-Azure-Maps-Subscription-key}&api-version=1.0&language=en-US&query=400 Broad St, Seattle, WA 98109
    ```

3. Click the blue **Send** button. The response body will contain data for a single location.

4. Now, we'll search an address that has more than one possible locations. In the **Params** section, change the `query` key to `400 Broad, Seattle`. Click the blue **Send** button.

    :::image type="content" source="./media/how-to-search-for-address/search-address.png" alt-text="Search for address":::

5. Next, try setting the `query` key to `400 Broa`.

6. Click the **Send** button. You can now see that the response includes responses from multiple countries. To geobias results to the relevant area for your users, always add as many location details as possible to the request.

## Fuzzy Search

[Fuzzy Search] supports standard single line and free-form searches. We recommend that you use the Azure Maps Search Fuzzy API when you don't know your user input type for a search request.  The query input can be a full or partial address. It can also be a Point of Interest (POI) token, like a name of POI, POI category or name of brand. Furthermore, to improve the relevance of your search results, the query results can be constrained by a coordinate location and radius, or by defining a bounding box.

>[!TIP]
>Most Search queries default to maxFuzzyLevel=1 to gain performance and reduce unusual results. You can adjust fuzziness levels by using the `maxFuzzyLevel` or `minFuzzyLevel` parameters. For more information on `maxFuzzyLevel` and a complete list of all optional parameters, see [Fuzzy Search URI Parameters].

### Search for an address using Fuzzy Search

In this example, we'll use Fuzzy Search to search the entire world for `pizza`.  Then, we'll show you how to search over the scope of a specific country. Finally, we'll show you how to use a coordinate location and radius to scope a search over a specific area, and limit the number of returned results.

>[!IMPORTANT]
>To geobias results to the relevant area for your users, always add as many location details as possible. To learn more, see [Best Practices for Search].

1. In the Postman app, select **New** to create the request. In the **Create New** window, select **HTTP Request**. Enter a **Request name** for the request.

2. Select the **GET** HTTP method in the builder tab and enter the following URL. For this request, and other requests mentioned in this article, replace `{Your-Azure-Maps-Subscription-key}` with your Azure Maps subscription key.

    ```http
    https://atlas.microsoft.com/search/fuzzy/json?&api-version=1.0&subscription-key={Your-Azure-Maps-Subscription-key}&language=en-US&query=pizza
    ```

    >[!NOTE]
    >The _json_ attribute in the URL path determines the response format. This article uses json for ease of use and readability. To find other supported response formats, see the `format` parameter definition in the [URI Parameter reference] documentation.

3. Click **Send** and review the response body.

    The ambiguous query string for "pizza" returned 10 [point of interest result] (POI) in both the "pizza" and "restaurant" categories. Each result includes details such as street address, latitude and longitude values, view port, and entry points for the location. The results are now varied for this query, and are not tied to any reference location.
  
    In the next step, we'll use the `countrySet` parameter to specify only the countries/regions for which your application needs coverage. For a complete list of supported countries/regions, see [Search Coverage].

4. The default behavior is to search the entire world, potentially returning unnecessary results. Next, we’ll search for pizza only the United States. Add the `countrySet` key to the **Params** section, and set its value to `US`. Setting the `countrySet` key to `US` will bound the results to the United States.

    :::image type="content" source="./media/how-to-search-for-address/search-fuzzy-country.png" alt-text="Search for pizza in the United States":::

    The results are now bounded by the country code and the query returns pizza restaurants in the United States.

5. To get an even more targeted search, you can search over the scope of a lat./lon. coordinate pair. In this example, we'll use the lat./lon. of the Seattle Space Needle. Since we only want to return results within a 400-meters radius, we'll  add the `radius` parameter. Also, we'll add the `limit` parameter to limit the results to the five closest pizza places.

    In the **Params** section, add the following key/value pairs:

     | Key | Value |
    |-----|------------|
    | lat | 47.620525 |
    | lon | -122.349274 |
    | radius | 400 |
    | limit | 5|

6. Click **Send**. The response includes results for pizza restaurants near the Seattle Space Needle.

## Search for a street address using Reverse Address Search

[Get Search Address Reverse] translates coordinates into human readable street addresses. This API is often used for applications that consume GPS feeds and want to discover addresses at specific coordinate points.

>[!IMPORTANT]
>To geobias results to the relevant area for your users, always add as many location details as possible. To learn more, see [Best Practices for Search].

>[!TIP]
>If you have a set of coordinate locations to reverse geocode, you can use [Post Search Address Reverse Batch] to send a batch of queries in a single request.

In this example, we'll be making reverse searches using a few of the optional parameters that are available. For the full list of optional parameters, see [Reverse Search Parameters].

1. In the Postman app, select **New** to create the request. In the **Create New** window, select **HTTP Request**. Enter a **Request name** for the request.

2. Select the **GET** HTTP method in the builder tab and enter the following URL. For this request, and other requests mentioned in this article, replace `{Your-Azure-Maps-Subscription-key}` with your Azure Maps subscription key. The request should look like the following URL:

    ```http
    https://atlas.microsoft.com/search/address/reverse/json?api-version=1.0&subscription-key={Your-Azure-Maps-Subscription-key}&language=en-US&query=47.591180,-122.332700&number=1
    ```

3. Click **Send**, and review the response body. You should see one query result. The response includes key address information about Safeco Field.
  
4. Now, we'll add the following key/value pairs to the **Params** section:

    | Key | Value | Returns
    |-----|------------|------|
    | number | 1 |The response may include the side of the street (Left/Right) and also an offset position for the number.|
    | returnSpeedLimit | true | Returns the speed limit at the address.|
    | returnRoadUse | true | Returns road use types at the address. For all possible road use types, see [Road Use Types].|
    | returnMatchType | true| Returns the type of match. For all possible values, see [Reverse Address Search Results].

   :::image type="content" source="./media/how-to-search-for-address/search-reverse.png" alt-text="Search reverse.":::

5. Click **Send**, and review the response body.

6. Next, we'll add the `entityType` key, and set its value to `Municipality`. The `entityType` key will override the `returnMatchType` key in the previous step. We'll also need to remove `returnSpeedLimit` and `returnRoadUse` since we're requesting information about the municipality.  For all possible entity types, see [Entity Types].

    :::image type="content" source="./media/how-to-search-for-address/search-reverse-entity-type.png" alt-text="Search reverse entityType.":::

7. Click **Send**. Compare the results to the results returned in step 5.  Because the requested entity type is now `municipality`, the response does not include street address information. Also, the returned `geometryId` can be used to request boundary polygon through Azure Maps Get [Search Polygon API].

>[!TIP]
>To get more information on these parameters, as well as to learn about others, see [Reverse Search Parameters].

## Search for cross street using Reverse Address Cross Street Search

In this example, we'll search for a cross street based on the coordinates of an address.

1. In the Postman app, select **New** to create the request. In the **Create New** window, select **HTTP Request**. Enter a **Request name** for the request.

2. Select the **GET** HTTP method in the builder tab and enter the following URL. For this request, and other requests mentioned in this article, replace `{Your-Azure-Maps-Subscription-key}` with your Azure Maps subscription key. The request should look like the following URL:
  
    ```http
    https://atlas.microsoft.com/search/address/reverse/crossstreet/json?&api-version=1.0&subscription-key={Your-Azure-Maps-Subscription-key}&language=en-US&query=47.591180,-122.332700
    ```

    :::image type="content" source="./media/how-to-search-for-address/search-address-cross.png" alt-text="Search cross street.":::
  
3. Click **Send**, and review the response body. You'll notice that the response contains a `crossStreet` value of `South Atlantic Street`.

## Next steps

> [!div class="nextstepaction"]
> [Azure Maps Search service](/rest/api/maps/search)

> [!div class="nextstepaction"]
> [Best practices for Azure Maps Search service](how-to-use-best-practices-for-search.md)

[Search service]: /rest/api/maps/search
[Route]: /rest/api/maps/route
[Weather]: /rest/api/maps/weather
[Search Address]: /rest/api/maps/search/getsearchaddress
[Fuzzy Search]: /rest/api/maps/search/getsearchfuzzy
[Reverse Address Search]: /rest/api/maps/search/getsearchaddressreverse
[Search Address Reverse Cross Street]: /rest/api/maps/search/getsearchaddressreversecrossstreet
[Azure Maps account]: quick-demo-map-app.md#create-an-azure-maps-account
[subscription key]: quick-demo-map-app.md#get-the-primary-key-for-your-account
[Postman]: https://www.postman.com/
[Get Search Address]: /rest/api/maps/search/getsearchaddress
[Post Search Address Batch]: /rest/api/maps/search/postsearchaddressbatch
[Fuzzy Search URI Parameters]: /rest/api/maps/search/getsearchfuzzy#uri-parameters
[Get Search Address Reverse]: /rest/api/maps/search/getsearchaddressreverse
[point of interest result]: /rest/api/maps/search/getsearchpoi#searchpoiresponse
[Post Search Address Reverse Batch]: /rest/api/maps/search/postsearchaddressreversebatch
[Reverse Search Parameters]: /rest/api/maps/search/getsearchaddressreverse#uri-parameters
[Best Practices for Search]: how-to-use-best-practices-for-search.md#geobiased-search-results
[Road Use Types]: /rest/api/maps/search/getsearchaddressreverse#uri-parameters
[Reverse Address Search Results]: /rest/api/maps/search/getsearchaddressreverse#searchaddressreverseresult
[URI Parameter reference]: /rest/api/maps/search/getsearchfuzzy#uri-parameters
[Search Coverage]: geocoding-coverage.md
[Search Polygon API]: /rest/api/maps/search/getsearchpolygon
[Entity Types]: /rest/api/maps/search/getsearchaddressreverse#entitytype
