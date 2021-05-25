## Objective 

Direct users through an SMP-native painpoint/stakeholder annotation process that allows them to respond to map content. 

The raw response data for each annotation is stored in triples like this:

| Subject | Predicate | Object |
|---------|-----------|--------|
| annotation IRI | creator | joe.bloggs@example.com |
| annotation IRI | hasTarget | stakeholder/painpoint IRI |
| annotation IRI | hasBody | "Some comment text" |
| annotation IRI | hasCategory | "Surprising" |

This can be easily aggregated into an output table with the following structure, accessed via API:

| Participant | Object | Category | Comment |
|-------------|--------|----------|---------|
| joe.bloggs@example.com | Example Painpoint | Surprising | null |
| joe.bloggs@example.com | Example Stakeholder | Question | "blahblahblah" |
...

This may then be further aggregated as desired -- for example, into an equivalent data format as the table in the existing Powerpoint doc.

## Process

### User identity

A user is authorised to access a map by including their email address (or a wildcarded version of it) in the whitelist for the map's project.

The user is emailed a link to the map. When they access it, they are first prompted to enter their email email. This is checked against the whitelist for access, and will also be used as their identity for entering annotations.

### Explaining the exercise to user

When the user accesses the map for the first time, they are greeted with an Introduction splash outlining the exercise and explaining how to leave annotations (categories and comments) on painpoints/stakeholders.

### Leaving annotations

Clicking on a painpoint brings up the annotation interface. A user can assign a category to a painpoint (Interesting/Surprising etc.) and optionally include a comment. (Emptystring if not entered?)

The painpoint annotation data goes into the UserMapAnnotationBag (a database table) for that user. Each UserMapAnnotationBag is associated with a single user and a single map.

### Output data

An API is called which aggregates all of the UserMapAnnotationBags for the target map together, and returns a tabular data file similar to the one outlined in the objective. 

## Questions:

Q: The MVP is an API that returns a dataset similar to that contained within the existing Powerpoint doc. Is this sufficient? 

A: *Yes, as MVP*.

Q: Users can currently make unlimited annotations. Is it necessary to limit the number of annotations which can be made so that feedback can be focused/prioritised? 

A: *No, it's fine for them to make as many annotations as they want*.

Q: Is being able to assign just a category to a painpoint (without associated comment) a necessary piece of functionality for this exercise? 

A: *Ideally the feedback process is as streamlined as possible (emoji-style), so the ability to leave it via a two- or three-click process of selecting a stakeholder/painpoint and then selecting a category without having to enter a full comment would be very nice.*


