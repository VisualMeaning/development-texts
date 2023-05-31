# Framing for selection

Thinking about how selection and changes to selection are shown.

### Selection

There can be one 'selected' item. This must be something in the model identified by an IRI, and then appears in the 'hash' portion of the url.

The selected item panel reacts to the change of selection, and various aspects of the map presentation do as well - labels for that item have a (subtle, dark) border, edges may be shown, and so on.


### Framing

How the browser window presents the map pane. This is primarily the centre point and resolution of the map, both of which are adaptive based on screen size and shape. Experimentally, this can also include shading of an area related to the item, or some other way of marking map content.


## Expectations

The presented selection should be clear and meaningful. That means it should ideally have visible on screen all of the content related to the selection, and some of the surrounding context from the environment, such as the parent, and siblings. Content should not be obscured by other elements of the platform. For container-type items, what is contained within them should be clear, even if all descendants are not shown at that detail level. There's an intersection between map design, ontology, and behaviours derived from the model.


## Existing behaviours

Currently there are three distinct cases to consider:
* New map load with selection set
* Select new item from link outside the map
* Select new item on the map

The new map load does not animate, but does show the constrained portion of the map and any selection effects.

Selecting by a navigation link causes an animation from the current map location, ending up at the same position as a new load. It's similar to the leaflet flyto effect in that it includes some zoom as well as pan, but does not have an adaptive speed based on distance at present.

Selecting by something on the map, a marker or label, does not cause any animation or change to the position of the map. (Note that web mapping platforms do tend to animate in this case, at least some of the time, for example if selecting a placename, but not if selecting a marker for a location.)


## Bounds

Bounds for an item to zoom to is currently often a rough heuristic based on the vector export, which is a rectangle drawn around all of the content at that semantic level. This is a reasonable 80% solution, without a lot of manual work to add outlines in the illustrator file, but can have edge case surprising behaviours.


## What should change

Bounds are not intrinsic to an item, they are specific to an item in a lens. To update this, we need an equivalently good heuristic for what to show on screen for any selection.
