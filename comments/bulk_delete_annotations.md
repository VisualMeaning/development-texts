## API for bulk deletion of annotation bags

This is a developer-focused API for quickly deleting many UserMapAnnotationBags for a single user on a single map. It is useful to have this API because each new annotation made by a user creates a new UserMapAnnotationbag, and so one user can end up with dozens of them which are a pain to delete via the Django admin interface.

### How to call

In your browser, or via curl, hit the following endpoint:

    https://staging.ecosystem.guide/api/annotations/delete-for-map?map=<map_iri>

`<map_iri>` is the IRI for the map you want to delete annotations for. This call will delete all annotations for this map **for the currently logged in user** -- so if I (AL) am logged in as `andrew.lightwing@visual-meaning.com` and I call this on `map=vm:maps/eom-mgs-ds`, it will delete all annotations for `eom-mgs-ds` owned by `andrew.lightwing@visual-meaning.com`.

Note that the `map` parameter may be either the full IRI for a map -- `http://visual-meaning.com/rdf/maps/eom-mgs-ds` -- or a shorthand form we are calling a qname. Qnames rely on the fact that for most IRIs the first chunk of an IRI is boilerplate from IRI to IRI -- in the example here the interesting part of the IRI is the `maps/eom-mgs-ds` bit, while the preceding `http://visual-meaning/rdf/` part is uninteresting for the purposes of identifying a map. 

So, instead of passing `http://visual-meaning/rdf/` as part of every map IRI, we can instead shorthand this part of the IRI by substituting `vm:` in its place. This means that calling

    https://staging.ecosystem.guide/api/annotations/delete-for-map?map=vm:maps/eom-mgs-ds

is the same as calling

    https://staging.ecosystem.guide/api/annotations/delete-for-map?map=http://visual-meaning/rdf/maps/eom-mgs-ds

### Deleting annotations for users other than the current user

For most development/testing purposes deleting annotations for just the current user (i.e. yourself) is sufficient. However, guest users (or anonymous users) who have given us an email identity via the `guest-email` entry point but who are not logged in with full user accounts may also make annotations. To delete annotations made in the course of testing annotation functionality for these guest users, an optional `user_email` parameter may also be passed.

    https://staging.ecosystem.guide/api/annotations/delete-for-map?map=<map_iri>&user_email=<user_email>

This will delete all annotations for this specific user for the given map.

**You must have DELETE permissions for UserMapAnnotationBags in Django in order to use this feature.** If you do not have this permission, attempting to pass `user_email` as part of this API call will result in a 401 Unauthorized response.