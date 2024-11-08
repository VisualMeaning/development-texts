
# URL Design on Shared Meaning Platform

## Sample URL

https://staging.ecosystem.guide/maps/mgs-ds/hr?plottable=direct&s=in&a=%21Stakeholder&kind=capitalTypes%2Fhuman&rel=_crossmarket-theme_human-resources+%21_stakeholder_3pl

## Overview

A URL (Uniform Resource Locator) is an address that specifies the location of a unique resource on the internet. In our application, URLs are crucial for sharing data and specific views between different clients. Here, we’ll explore the structure and role of each URL component within our platform.

## General URL Structure

The typical structure of a URL is as follows:

scheme://domain/path?query_parameters#fragment


### URL Components:

- **Scheme**: This part (e.g., `https`) defines the protocol used, typically `http` or `https` for websites.
- **Domain**: The main address (`staging.ecosystem.guide`) points to the server.
  - Different environments may be specified by subdomains.
- **Path**: This part (`/maps/mgs-ds/hr`) defines the page route within the React app, determining which page and components should render on the client.
- **Query Parameters**: The section after the `?` (`plottable=direct&s=in&a=%21Stakeholder&kind=capitalTypes%2Fhuman&rel=_crossmarket-theme_human-resources+%21_stakeholder_3pl`) contains key-value pairs that pass additional data to the client.
- **Fragment (optional)**: Identified by the `#` symbol, this part points to a specific component, tab, or content section on the page, potentially scrolling to or highlighting a view in the component.

## Detailed URL Breakdown on the Shared Meaning Platform

On our platform, each part of the URL is thoughtfully designed to represent a specific state or view within the application. This allows users to not only navigate directly to various sections but also to capture and save specific views or data states. Users can then share these URLs with others, enabling consistent, shared access to the same content or settings across different clients.

### Path

**Example Path**: `/maps/mgs-ds/hr`

The path consists of several components, each controlling navigation within the application. They should be in exactly this order:

1. **Page** (`/maps/`): Directs the user to either the main map page or the map terms page, where a list of all map terms is displayed.
   - Possible values: `maps`, `mapterms`
2. **Map** (`/mgs-ds/`): Represents the map path, defining which map data to display.
3. **View (optional)** (`/hr/`): Represents a specific view. Each map may contain multiple views, and this part controls which one is shown.
- **Note**: Some older maps, such as `/maps/lotus`, do not have views, but views are required for all new maps.

### Query Parameters (in order of appearance)

Query parameters are key-value pairs appended to the URL after the `?` symbol. They serve two main types of functions in our app:

1. **Domain Type Query Parameters**  
These parameters define the map's domain, such as region or type, and act as their own filtering layer. Unlike other filters, domains work with a logical AND in combination with other domains and filters. Domains can also support logical OR with the use of the `+` symbol.  
   - **Examples**:
     - `domain-Force=_police_force+military_force` – In this case, the domain is multi-domain, meaning it includes both `police_force` and `military_force` as separate filters.
     - `domain-Region=_Brazil+_China+_Russia`
     - `domain-BusinessService=_business-service_crime-prevention-service`
     - `domain-PoliceForce=_police-force_leicestershire`

2. **Filter Type Query Parameters**  
   These parameters control data filtering on the page. They can be reset via a filter reset button and are set dynamically.

   - **plottable**: Controls the plot settings, allowing values such as `any`, `related`, `direct`, and `detached`.
   - **s**: Filters items containing the specified text (string).
   - **a**: Filters items by including or excluding specified classes. Multiple classes are concatenated with a `+` symbol, indicating a union (not an intersection). To exclude a class, `%21` (encoded `!`) is used.
     - Example: `a=Brightspot+Category` includes items with either the "Brightspot" or "Category" class.
     - Example: `a=%21Brightspot` excludes items with the "Brightspot" class.
   - **rel**: Filters only items that are related to other items. Supports concatenation with `+` and exclusion with `%21`.
     - Example: `rel=_painpoint_gp_tricky-patients+_stakeholder_capability`
   - **kind**: Filters by category type, supporting inclusion and exclusion.

3. **Grouping and Sorting** – Used for the list view:
   - `group=`: Possible values `type`, `org`, `category`, and `stakeholder`
   - `sort=`: Possible value `order`

### Fragment (always at the end of the URL)

**Example**: `#_stakeholder_distributor-owners`

In our application, the fragment (following the `#`) always appears at the end of the URL and serves a specific purpose that differs slightly from traditional URL fragment behavior. Unlike typical web applications where the fragment points to an HTML element's `id`, in our platform, fragments refer to data elements with their own unique identifiers, known as IRI (Internationalized Resource Identifiers).

The fragment serves two main purposes:

1. **Navigating to Data Elements**: The fragment helps reference a specific data element in the application, which may not correspond directly to a static HTML element but to a data item identified by its IRI. For example, `#_stakeholder_distributor-owners` refers to a data item identified by the IRI.
   
2. **Item Reference**: Fragments are used to create direct references to data items, enabling precise navigation to and interaction with specific data within the application. These IRIs do not include namespaces (prefixes), which are typically handled by `rdf.js` in our system. The fragment values are stripped of namespace prefixes and contain only the part of the IRI relevant to our platform’s identifiers.

**Note**: This behavior applies to all values in the query parameters, which are also based on IRI identifiers without namespaces.