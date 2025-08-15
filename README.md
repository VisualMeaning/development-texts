# Visual Meaning development texts

Collection of development-led text documents related to technical decision making, other writeups, and anything else that shouldn't go to sharepoint to die.

> [!NOTE]
> This repository is public. Developer documentation containing sensitive information that should not be public-facing should be versioned in the [development-texts-private](https://github.com/VisualMeaning/development-texts-private) repository.


## Comments

* [Rethinking zoom](comments/rethinking_zoom.md) - follow up proposal required for design authority.
* [Implementing Basic Web Analytics for the SMP](comments/analytics_summary.md) - objectives and a summary of options
* [Backing up the Postgres Databases in Heroku](comments/heroku_pgs_backups.md)
* [Stakeholder annotations](comments/stakeholder_annotations.md)
* [Annotation bulk deletion API documention](comments/bulk_delete_annotations.md)
* [Tile upload and data ingest notes](comments/data_ingest_gotchas.md) - Notes on various dos/don't dos of the data upload process.
* [Domain hosting](comments/domain_hosting.md) - an overview of domain hosting options as we want to migrate away from our current provider
* [Panels](comments/panels.md) - panel components and layout.
* [Reduce map loading times](comments/loading_perf.md) - notes on potential actions we can take to improve the load times of maps with large amounts of model data.
* [Manually generating SSL certs for Azure web apps using Certbot](comments/certbot_ssl_for_azure_web_app.md) - how to generate and deploy free Let's Encrypt certs to an App Service running in Azure
* [Notes from manual db backup/restore in Azure](comments/db_manual_notes.md) - Use `pg_dump`/`pg_restore` to manually take and restore backups from a Postgres DB running in Azure
* [Using pyenv to manage Python versions](comments/pyenv_and_python.md) - Step-by-step instructions for installing and using pyenv for Python version management.


## Records

* [Mapping libraries](records/mapping_libraries.md) - decided to use [Openlayers](https://openlayers.org/).
* [RDF in Postgres](records/postgres_rdf_json_querying.md) - decided to leave status quo for now.
* [Working with upstreams](records/working_with_upstreams.md) - proposed process for working with and funding upstream projects.
* [Process of adding a new string using the Translation component in sm_platform](records/SMP_adding_translation.md)
* [Accessibilities in sm_platform](records/accessibilities.md)
* [Improving search](records/improving_search.md) - deciding to try out one or two client-side search libraries
* [Showing edges](records/showing_edges.md) - thoughts on how to draw edges between markers and labels in our maps
* [Working with upstreams](records/working_with_upstreams.md) - working practice/process for when and how to work with non-VM upstream repositories


## Postmortems

* [2020-08-18 PC revert](postmortems/2020-08-18_pc_revert.md)
* [2021-05-03 ecosys timeouts](postmortems/2021-05-03_ecosys_timeouts.md)
* [2023-03-20 Azure dev outage](postmortems/2023-03-20_azure_dev_outage.md)
* [2024-03-22 Azure dev deployment week long outage](postmortems/2024-03-22_azure_dev_outage.md)
