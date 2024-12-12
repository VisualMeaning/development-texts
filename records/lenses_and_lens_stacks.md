# Lenses and Lens Stack

Lens is a common language where development can go so frontend can behave in consistent way.
Sets of tiles and labels and their associated data can be considered lenses. We've also described these to consultants as being a focus on a slice of a model.

## Understanding the Problem with Layer Blending

#### Context-Dependent Layers
Normal maps consist of multiple togglable layers (e.g., rivers, streets) that users can activate or deactivate as needed.
These layers often lack a fixed order and rely on user decisions to make sense within the current context.
#### Blending Challenges
Layers must be semi-transparent and blend seamlessly to avoid visual clutter or misrepresentation.
For example, comparing different datasets, such as maps from 2020 and 2024, often requires additive blending to highlight changes.
A generalized blending model (e.g., lenses) requires prior knowledge of the context to ensure the resulting visualization is meaningful and coherent.


## Tree Structure for Lenses
The tree structute for lenses have certain things in mind:
* **Hierarchical Organization** - The tree structure establishes a clear base layer (e.g., the foundational map) and organizes other layers in a logical hierarchy.
Each branch or node in the tree reflects a contextual relationship to the base layer, ensuring that additional layers make sense within that specific spatial or temporal framework.
* **Contextual Relevance** - Example: In comparing 2020 to 2024 maps, certain annotations (e.g., arrows indicating trends) might be relevant for 2020 but irrelevant or misleading for 2024.
By predefining relationships between layers, the tree structure reduces ambiguity for users, ensuring that lenses applied to the map yield meaningful insights.
* **Encouraging Better Design Practices** - The tree structure requires map creators to think critically about the dependencies and interactions between layers during the design phase.
This proactive approach avoids confusion and ensures a consistent and intuitive user experience.
The structure allows for flexibility in adding or modifying layers while maintaining a coherent framework, making it easier to adapt to evolving contexts or datasets.



## Some important definitions

Please note that a number of these are the definitions proposed in the Platform Glossary by MN. 

### Geography
'Geography' has no concrete implementation in the platform, where as maps, lenses, detail levels all have code. It's a conceptual tool for helping us think about how we use maps and lenses.

The embodied spatial metaphor* of a map, borrowing design tropes from geographic maps, provides a context for arranging abstract concepts, making them concrete and graspable, and their relations meaningful.

Where lenses share geography, this is effectively a promise that an element will be found in the same location (at the same X Y cordinates) within that space.

By "the embodied spatial metaphor of a map" we just mean using geographic map design tropes to help make sense of client content (target domain) in terms of physical objects in space (source domain). It's metaphorical in that the space represented in the map is not 'real' in the geographic 'map of the world' sense, but applies to relationships between concepts.  We all have bodies and lived experience of interacting bodily with the world, through which we share common intuitive understandings for various configurations of physical objects. This is the basis for using maps to create shared meaning.

## Lenses in Adobe Illustrator & Semantic Tools


Then concept of a lens on the Shared Meaning Platform must translate to how maps are created in and exported from Adobe Illustrator. 

### Layer / detail level

Layers are a concept in Adobe Illustrator, that their [help docs](https://helpx.adobe.com/lv/illustrator/how-to/ai-layers-basics.html#:~:text=Layers%20are%20like%20transparent%20folders,stack%20in%20the%20Layers%20panel.) describe as being:
* "Layers are like transparent folders for managing content"
* "Layers contain the images, text, or artwork that make up a layered file" - for us, this specifically means vignettes, platforms, and labels
* "Layers are arranged in a stack in the Layers panel. In the document, content on the layers at the top of the Layers panel appears in front of content on layers that are lower in the panel" (This is also why consultants use the term "overlay" - it refers to a layer that appears higher in the layers panel than another, where the two layers are meant to be seen together with the one "over" the other)
* Layers can also be hidden, locked, and moved in useful ways. 

MN has also described these as "a named top level grouping of graphical objects that can be toggled on and off. These get exported individually with some elements, such as labels, extracted from the graphics and presented in json alongside."

For a given Adobe Illustrator file that is a map destined for the Shared Meaning Platform, **a layer which has been made semantic is a detail level.** For a layer to be semantic, it must have had '-~' inserted into the front of its layer name so that it can be picked up by the Semantic Tools.

### Set of detail levels
A set of detail levels which share the same geography and also have consistency of platforms remaining in the same place should be designed to create the illusion of zoom. 

That is to say, a detail level of the lowest number (1) should contain the lowest amount of detail (fewest number of vignettes and labels, and most minimal visual complexity) and appear to be the most zoomed out version of the set of detail levels. It should be assumed that the least detailed detail level will be viewed as a whole.

A detail level of the highest number should contain the highest amount of detail (all of the vignettes and labels possible in the set and the most visual complexity) and appear to be the most zoomed in version of the set of detail levels. The most detailed detail level should also not be expected to be viewed as a whole, and only as parts where users are zoomed in. 

The least detailed detail level should not contain any label that is not represented on more detailed levels. Those labels may be hidden, but they should be present. 

**A complete set of detail levels makes up a lens.**

### Lens

MN proposed the following definition for lens: 
"A unit of content at the level above a detail level and beneath a map, which can be spatially and temporally related or unrelated in various ways which need to be specified. Why the name 'Lens'? A lens (physical object) is transparent; you look through it at stuff. Our lenses are structured presentations of model content with visual language applied in order to make meaningful distinctions between dimensions of the data. So a lens is never, in itself, displayed; we look at a view of model content, as it were, through a lens."

"Where lenses represent the same conceptual space, they should share geography."

### Set of lenses / Lens stack

**A set of lenses may make up a map.** For this to be true, they must share geography, but might not share the same platforms.

A set of lenses that form a map might have different platform locations due to a change in time state causes the platforms to change.
Differing lenses for the same map might cover different sub-sets in a recursive population, or introduce different scenarios for the same geography. 

The current implementation of the Shared Meaning Platform makes little room for a Map (as a thing which appears in the "Map Switcher") to be anything other than a single lens (although this may be a child lens together with its parent/s).

For lenses to be in a set that makes up a map, typically there will be parent and child relationships between the lenses. 

A parent lens should represent all the structure and context which does not change for the children. 
Child lenses may only add information (i.e. they may not subtract it) because they inherit lenses from their parents. They do not necessarily need to inherit tiles / visuals from their parents, although the current expectation captured in the Semantic Export Tooling is that children are overlays, and do expect to inherit tiles / visuals from their parents. Due to the nature of overlay visualization, a set of lenses can also be called a lens stack, where the child lens visuals sit on top of its parent's down until the greatest parent.

Improvements to Semantic Tooling would allow for an indication of whether or not something is an overlay in addition to being a child lens. 

Sets of lenses and detail levels are expected (but not required) to share an Artboard (and set of co-ordinates) in Adobe Illustrator, but the mere sharing of an artboard in Adobe Illustrator does not meet the threshold for the definition of a shared "geography".

### Splitting Across Adobe Illustrator Files

In theory, the detail levels that make up a lens should be contained in the same Adobe Illustrator file, but multiple lenses that make up a map should be able to be split across multiple Adobe Illustrator files and a set of maps that make up a client project that may appear on one Django map should also be able to be across multiple Adobe Illustrator files. 

Unfortunately, the Semantic Tools are not currently able to indicate parent / child relationships for lenses across Adobe Illustrator files, so a full map should appear in Adobe Illustrator file. 
Improvements to the Semantic Tools would address this to enable file sizes / complexity to be adequately managed. 

A map IRI is inserted in a field called "Project" in the Semantic Tree Tools panel, so multiple maps that belong to the same client project and should appear on the same map IRI can be indicated across multiple Adobe Illustrator files. 

### Zoom levels

Zoom level is separated from detail level as they comply to the devices capability rather than model.
_Do zoom levels need some kind of definition or starting explanation here?_
_AD may wish to check this_

Detail levels have a strong relationship with zoom levels in the current implementation of the Shared Meaning Platform, whereby a given detail level is set to apply one, or a range of, zoom level/s. Often (although it is not a requirement) they may be set as follows: 
Zoom levels 0 - 2 for Detail Level 1
Zoom levels 3 - 4 for Detail Level 2
Zoom levels 5 - 6 for Detail Level 3

The full range for zoom levels for a map is (at present) 0 - 7, but this is clamped to 0 - 6 by default. Manual specification in the job file can set the maxZoom to 7 or a number less than 6.  

Detail levels must sequentially apply to the zoom levels. 

It would not be permissiable for detail levels from the same lens to apply to the same zoom level (although this is expected for a parent and child lens to each have a detail level applied to a given zoom level).

Zoom levels have increasing visual fidelity of elements the higher the number goes. As such the geophraphy of the carpet where all elements sit on also has increased fidelity tied to resolution.

The large geography image is divided into smaller tiles that load concurrently, allowing users to view sections of the map progressively without waiting for the entire image to load. This helps accommodate cognitive load considerations and improves user experience by minimizing waiting time.

Tiles have a fixed dimension of 256x256. Each zoom level resolution increases by a constant factor for consistent change perception between every two neighbouring zooms.

| **Zoom level number** | **Total number of tiles** | **Number of columns and rows of tiles** | **Total resolution (px)** |
| ---: | ---: | ---: | ---: |
| 0 | 1 | 1 | 256 x 256 |
| 1 | 4 | 2 | 512 x 512 |
| 2 | 16 | 4 | 1,024 x 1,024 |
| 3 | 64 | 8 | 2,048 x 2,048 |
| 4 | 256 | 16 | 4,096 x 4,096 |
| 5 | 1,024 | 32 | 8,192 x 8,192 |
| 6 | 4,096 | 64 | 16,384 x 16,384 |
| 7 | 16,384 | 128 | 32,768 x 32,768 |

#### Issues with the current implementation of zoom levels
In the current implementation of the Shared Meaning Platform, the resolution of a monitor that a map is displayed on determines which zoom level a user lands on when viewing the map. Annecdotally, most laptop users with an HD screen will land on zoom level 2. 

If a detail level has only been applied to zoom levels 0 and 1, then most of our users (on the assumption that an HD laptop is standard issue for most companies) will not see the detail level applied to those zoom levels. This is because, anecdotally, most laptop users with an HD screen will land on zoom level 2, and therefore not see zoom levels 0-1 unless they manually zoom all the way out.

#### Plans for dynamic zoom level switching based on user display size

To improve implementation of zooming on the platform, detail lenses would become the dominant experience, and would be available at all resolutions necessary for a user's screen, so that the user experience of landing on a map and zooming in to it is not altered by the resolution of the user's monitor. 

## Semantic Tools implementation of lenses

In Adobe Illustrator, users can name layers manually, but the Semantic Tools will also rename them if needs be. 

First, a lens is created and named from the Semantic Tree Tools panel.
In the current implementation of the Semantic Tools, the name may not contain dashes or spaces and may only make use of underscores. This is annoyingly inconsistent with the remainder of the map upload process, which assumes terms in a lens name are separated by dashes. This would ideally be remedied. 

Then, layers in Adobe Illustrator may be attributed to that lens to become defined as detail levels. The layers should be added to the lens in order of least detailed to most detailed. 
This will cause the layers to be renamed in the following format: -~<lens_name>@<detail-level-number>. 

This process may cause errors if the artboard of the layer is not square or the colour profile is not set to RGB. 

Manually naming layers in the same way from the Adobe Illustrator Layers panel will cause them to be recognised as lenses by the Semantic Tools, provided the detail levels are numbered sequentially starting from 1. Other manual approaches will either cause errors or not be recognised at all. 

Non-semantic layers may exist in the Adobe Illustrator file, but will be ignored by the Semantic Tools and semantic export process. 

Lenses and the detail levels that make them up are shown in the Semantic Tree Tools panel. 

Lenses can have a parent attributed to them in the Semantic Tree Tools panel. 

Various tags can be assigned to each label, or they can be made unsemantic, in the Semantic Tagging Panel. 
Other objects can also have semantic tags applied - such as to be set as areas or data collection lines or to be set as not semantic at all.

Attributes such as fullName, anchorName, Class, kind etc. may also be added to semantic labels in the Semantic Attributes panel. 

## Challenges

### Conceptual Chalenges
A primary challenge we are addressing is ensuring that map content remains both accessible and coherent, regardless of detail level, detail resolution, or active lenses. Users need to locate and interpret information seamlessly, even when specific features aren’t visible due to their current view. For instance, a user should be able to search for a feature and navigate to its location even if it’s not currently displayed because of detail-level constraints or panning.

Another layer of complexity arises from the need to maintain visual and semantic coherence as users toggle between lenses. Each lens may introduce new content or modify the view, which can lead to overlapping or misaligned features. This raises the question: do the layers and lenses work together to provide a clear, meaningful narrative across varying levels of detail? For example, when comparing two maps (e.g., 2020 vs. 2024), semantic details like annotations or trends must align with the specific context of each version to avoid confusion.

A tree structure provides a solution by organizing lenses hierarchically and logically. It helps define clear relationships between layers, ensuring that features exist conceptually within the lens even if they are temporarily hidden due to detail level or other constraints. This approach allows for short and descriptive naming conventions that encapsulate two critical aspects: what is being rendered (resolution, details, tiles) and what conceptual elements are present (features tied to the lens but not necessarily visible in the current view). This abstraction ensures that users can navigate seamlessly across the map, guided by the conceptual presence of features rather than their immediate visibility.

However, implementation introduces challenges. Authors, for example, are often accustomed to the freedom of toggling layers on and off during map creation. Requiring structured organization or predefined naming conventions can feel like an unnecessary restriction. Balancing this need for structure with the flexibility authors expect is essential to avoid resistance and ensure adoption. Additionally, while much effort has gone into using layers and geographical lenses to convey detail, more interactive methods could be explored. We kind of have the Selected Item Panel allowing users to navigate parent-child relationships within the tree, but what can we do more?

Finally, user experience must remain at the center of this design. Visual clarity is critical, especially when balancing the density of information at different detail levels. Adhering to principles like "seven items per screen" can help tie details to content in a way that reduces cognitive overload, moving beyond rigid grid-like tile designs to adapt content dynamically based on user interaction.

By employing a tree structure, we address the fundamental need for clarity and usability while maintaining the flexibility authors require. This approach ensures that users can find and interpret map features intuitively, while also supporting more robust and interactive design possibilities.

### Practical Challenges in Map Design
One of the significant challenges lies in handling labels across zoom levels. Older maps often featured labels that were large and prominent when zoomed out, but as users zoomed in, the label size shrank, and its position shifted. While the label's general region remains the same, its precise location can vary significantly, making it difficult to provide a consistent user experience. Fallback solutions, such as landing at a fixed zoom level, often fall short. If the label’s position and size depend on the level of detail, users may encounter confusion about whether the label even exists at the current zoom level or where its “center point” truly lies.

This issue becomes pronounced in one of the suggestion for LECM. Highlighting a specific semantic or geographical area requires not the label's placement but also mechanisms to express that a feature exists in one region while its label is displayed in another. Creating tools that allow map authors to define such relationships systematically, rather than ad hoc, is essential for clarity and usability.

Another challenge is the conceptual approach to lenses. While they serve as a common language for frontend consistency, practical issues arise when a new feature or interaction is introduced without enough conceptual groundwork. Designing only at the functional level often leads to inconsistent and unpredictable results. For example, determining zoom interactions based on user input (“clicking on this item should zoom here”) may make sense intuitively but can become unmanageable when it requires manually defining thousands of specific areas without automated verification processes. This lack of systematic design risks creating expectations that are difficult, if not impossible, to meet.

Subclassifying map designs introduces constraints that must be balanced. For certain types of maps, intricate interactions like area coloring or detailed highlighting might be avoided altogether due to the absence of well-defined or meaningful area classifications. Features like “coloring areas by classification” are only effective when the areas themselves are sensibly delineated. Without a reliable foundation, such features risk creating outputs that are nonsensical or misleading.

### Work for Future Development
We need start some groundwork:
* The current approach ties detail levels closely to zoom levels. Decoupling these concepts would enable more flexible map interactions and allow creators to focus on semantic richness without being constrained by zoom-based representations.
* Much of the current map content relies on markers to represent information. However, placing markers effectively, particularly when they represent features not physically present on the map, requires more thoughtful design. Simply displaying all items, suppliers, or relationships results in cluttered and overwhelming visuals that fail to communicate effectively. Instead, better aggregation techniques are needed, using metadata patterns, shapes, or other commonalities to group information in ways that enhance understanding while reducing visual noise.
* Area-based features are necessary to make functionality like area bounding/coloring more systematic and reliable. For example, when showing classifications or highlighting specific areas, the system must rely on well-defined and meaningful area definitions. This involves not only fixing bugs but also ensuring that the data structure supports such interactions in a way that aligns with the map's overall conceptual integrity.


## Lens export assumptions and consequences: issues with lens stack and open layers
At present, semantic export triggered from the Semantic Tree Tools panel assumes that all lenses present in the file should be exported. The automated tiling pipeline assumes that a parent lens is not transparent and should have a white background, while a child lens is assumed to be transparent, so that it can be used as an overlay on the parent lens. 

The platform has concepts of parentLens, which also assumes that BOTH the tiles and labels from a child lens should appear together with the tiles and labels of a parent lens. While labels are valid, tiles were not originally conceived to be included by default. 

This has caused the situation where two (or more) sets of tiles appear together on a Map or View on the platform, and transparent background tiles are layered over one another on top of a set of parent tiles with a white background. This is inefficient from a loading perspective, error prone, brittle (it has caused issues when openlayers has been updated), and at odds with the original intentions for lenses. 

### Specify overlay and which parent tiles should be used

Improvements to the semantic tools would allow users to specificy whether a child lens is an overlay or not, and all of the parent lenses that is should appear over. 

### Introduce a compositor

Improvements to the tiling pipeline and/or Shared Meaning Platform could potentially introduce a compositor that would ensure that the tiles for a given child lens are one set that includes the visuals from both the child lens and any relevant parent lens. This would ensure:
* that only one set of tiles is ever relied on for display, and address a number of performance and loading issues.
* add a spacial elaboration for the child space, especially for overlays.

This would not address composition of lenses and labels, but only compositing tiles from stack of spaces.

## Lens Nodes for Domain-Aware Lenses
### Domains
Domains are Shared Meaning Platform specific parameters that provide another layer of content filtering. They are meant to be used as switching tool between states on the same geography. Such state for domains define contextual variability - properties or characteristics rather what the same context looks like over time.

The model classifies domains/instances of same type or characteristics into domain types. A combination of instances between domain types creates the context.

### Lens Nodes
While a piece of content may have several contexts, map objects (labels) may exist on only one space or one lens for each context.
Lens nodes or map of lens nodes are connectors between lenses pertaining to each context. Their function is to translate domain instance combinations to a single lens.

While still at model level, there is still no intention on baking in the semantic tooling and constructing them into the model can be a complicated manual process.


