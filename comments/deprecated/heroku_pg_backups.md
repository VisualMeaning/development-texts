> [!WARNING]
> This documentation is deprecated. We do not rely on Heroku's built-in backup/restore process and instead have our own process for capturing backups and shipping them to S3, which is [documented in the sm_platform repo](https://github.com/VisualMeaning/sm_platform/blob/master/scripts/Readme.md). Retaining this doc for historical interest only.

# Backing up the Postgres Databases in Heroku

The Heroku deployment of the SMP is backed by a PostgreSQL database that holds all of the Django model data for users, auth, projects, maps etc., and all of the map model data. If this database fails for whatever reason the SMP will go down, and if we are forced to restore from a backup we could potentially be looking at significant data loss and impact on the business.

We should therefore be taking automated backups of the database to minimise any potential data loss and make disaster recovery (relatively) easy. This document outlines what we are currently doing in this regard.

## Backing up via heroku pg:backups

### Manual backups

The Heroku platform has an integrated system for quickly taking backups of any running Postgres databases. These backups can be taken either by hitting a button in the Heroku web interface or (more usually) passing the following command to the Heroku CLI:

    heroku pg:backups:capture --app <app-name>

where **app-name** is `shared-meaning-staging` or `shared-meaning` depending on whether you're backing up staging or prod.

The backups are stored in Heroku's own storage layer, which is backed by Amazon S3. Backups can be manually downloaded to local storage if necessary.

### Automated backups

We are currently on the free tier of Heroku's Postgres attachment, which only allows two manual backups to be stored at once. However, it is also possible to schedule automatic backups via the Heroku CLI, and these are tracked separately from the manual backups. 

The automatic scheduling is outlined in more detail in the [Heroku documentation](https://devcenter.heroku.com/articles/heroku-postgres-backups#scheduling-backups), but the tl;dr is that all tiers of the service can currently schedule daily backups, and Heroku will store up to seven of these, giving us daily snapshots of the database for the last week.

Automatic backups are scheduled via the following command:

    heroku pg:backups:schedule DATABASE_URL --at '00:00 Europe/London' --app <app-name>

## The current state of things

**Automatic daily backups are currently running at midnight UTC on both staging and production.** In the event of needing to do disaster recovery for whatever reason we can restore to any of the daily snapshots taken in the last week, with the loss of no more than a day's worth of data. 

The low volume of updates to the SMP means a 24h interval between backups is acceptable for now -- the only significant user-authored data is annotations, and so our worst-case scenario is that we'd lose a day's worth of those. Any lost VM-authored content updates could be reconstructed from source data.

Ideally we would be storing the daily backups for up to a month, but retaining a week's worth is sufficient to give us a bare minimum level of DB resilience. 

## Further work

Further work could be done to investigate the possibility of shipping the backups from Heroku to our Amazon S3 setup, but some preliminary reading indicates this would be difficult if not impossible. Even upgrading the DB to a paid plan wouldn't give us more daily backups; going to a $50-per-month tier (see [pricing plans](https://elements.heroku.com/addons/heroku-postgresql)) would allow for storing four weekly backups, along with proper log-based rollbacks with a horizon of up to 4 days, and I'd recommend doing this before trying to bodge something on our own as the labour costs of figuring out a bespoke solution would comfortably exceed this for at least a half a year.
