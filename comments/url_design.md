# URL Design on Shared Meaning Platform

## Sample URL

https://staging.ecosystem.guide/maps/mgs-ds/hr?plottable=direct&s=in&a=!Stakeholder&kind=capitalTypes%2Fhuman&rel=\_crossmarket-theme\_human-resources+!\_stakeholder\_3pl

## Overview

A URL (Uniform Resource Locator) is an address that specifies the location of a unique resource on the internet. In our application, URLs are crucial for sharing data and specific views between different clients. Here, we’ll explore the structure and role of each URL component within our platform.

## General URL Structure

The typical structure of a URL is as follows:

scheme://domain/path?query=parameters#fragment

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

### Query Parameters (Overview)

Query parameters are key-value pairs appended to the URL after the `?` symbol and serve as the means of filtering data in our application. Below is a brief translation of how the filtering works:

- **`+` = OR**: Combines multiple values within a domain or filter type, allowing for an inclusive condition (logical OR).
- **`&` = AND**: Combines different query parameters, meaning that all conditions must be satisfied (logical AND).
- **Default behavior**: If no specific filter type or domain is provided, all instances of the type are accepted.

### Query Parameters (in order of appearance)

Query parameters are divided into two main categories: **Domain Type** and **Filter Type**. They allow users to dynamically control what data is displayed on the page.

#### 1. **Domain Type Query Parameters**  

These parameters define the map's domain, such as region or type, and act as a separate filtering layer.

- **Examples**:
  - `domain-Force=_police_force+military_force` – A multi-domain filter, including both `police_force` and `military_force` as separate filters.
  - `domain-Region=_Brazil+_China+_Russia`
  - `domain-BusinessService=_business-service_crime-prevention-service`
  - `domain-PoliceForce=_police-force_leicestershire`

<!--

As discussed as part of the extensions to domains work landing, the url
parameter should probably _just_ be the domain, and rely on uniqueness of the
domain individuals to resolve which domain types are operated on.

-->

#### 2. **Filter Type Query Parameters**  

These parameters control data filtering on the page and are dynamically set. They can be reset via a filter reset button.

- **plottable**: Controls the plot settings, with possible values such as `any`, `related`, `direct`, and `detached`.
- **s**: Filters items containing the specified search string.
- **a**: Filters items by including or excluding specified classes. Multiple classes are combined using the `+` symbol (union), while exclusion uses `%21` (encoded `!`).
  - Example: `a=Brightspot+Category` includes items with either the "Brightspot" or "Category" class.
  - Example: `a=%21Brightspot` excludes items with the "Brightspot" class.
- **rel**: Filters items that are related to other items, with support for concatenation (`+`) and exclusion (`%21`).
  - Example: `rel=_painpoint_gp_tricky-patients+_stakeholder_capability`
- **kind**: Filters by category type, supporting both inclusion and exclusion.

#### 3. **Grouping and Sorting**:

  - `group=`: Possible values are `type`, `org`, `category`, and `stakeholder`.
  - `sort=`: The only possible value is `order`.

These have been used in list view, but are not visually distinguished on the map.

### Fragment (always at the end of the URL)

**Example**: `#_stakeholder_distributor-owners`

In our application, the fragment (following the `#`) always appears at the end of the URL and serves a specific purpose that differs slightly from traditional URL fragment behavior. Unlike typical web applications where the fragment points to an HTML element's `id`, in our platform, fragments refer to data elements with their own unique identifiers, known as IRI (Internationalized Resource Identifiers).

The fragment serves two main purposes:

1. **Navigating to Data Elements**: Automatically zooms in on and centers a specific component within the Map view, highlighting it for easier identification and navigation.
   
2. **Item Reference**: Keeps the reference to the selected item that can be opened different panels in the application. This IRI do not include namespaces (prefixes), which are typically handled by `rdf.js` in our system. The fragment values are stripped of namespace prefixes and contain only the part of the IRI relevant to our platform’s identifiers.

**Note**: This behavior applies to all values in the query parameters, which are also based on IRI identifiers without namespaces.
