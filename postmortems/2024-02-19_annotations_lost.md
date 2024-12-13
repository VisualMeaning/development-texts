# 2023-02-19 Annotations not received/recorded by platform

## What
The opatlas deploy did not record 18 comments left by a user in the annotations feature and left no footprint of the issue. 

2023-02 (all times UTC)

* -19 (Monday) 15:24 Client user S begins recording comments using annotation feature - records 63 comments, can only see record of 45 comments after page refresh
* -19 (Monday) 17:11 Comments left after this time are the comments which have disappeared. POST requests match with timestamps in annotations bag. Hereafter, only GET requests. All 200 OK responses, no server errors. 
* -19 (Monday) 18:07 S emails consultant HM to inform VM that they had lost 18 comments and had re-entered the comments - indicated a suspicion they were lost on page refresh. No obvious loss of internet connection. 
* -20 (Tuesday) 09:30 (approx) HM raises first report during check-in. AL requests details to be forwarded.
* -20 (Tuesday) AL checks map term bags for comments left by user and shared timestamp for first recorded comment left with HM
* -21 Comments lost a second time; user confirms browser (Chrome), which map it occured on and other details
* -22 (Thursday) 15:32 HM reports second incident. HM and AL review the number of recorded comments and whether there were any duplications. HM requests more details from user. No evidence in platform log that missing 18 comments had been attempted to be sent.
* -22 (Thursday) 18:25 after discussion with MP, AL confirms his understanding of auto-save
    * When the user loads the platform in their browser it makes a GET request to the annotations API to get their current comments.
    * From this point on, comment state is tracked internally to the app, with no further GET request to the server.
    * The app has a boolean that says "Has my comments state changed since the last time I pushed comments to the server y/n"?
    * This boolean is checked every 15 seconds.
    * If the boolean is true (i.e. comments have been made or updated), a POST request is made to the annotations API to save the user's new comments state as a new annotations bag object.
    * The boolean is then set back to false. We think this might happen regardless of whether or not the POST request to save the new comments state was actually successful, and there are no retries in the event of failure.
    *   Therefore, it is possible that if a user loses internet connection while they're making comments, and doesn't re-establish that connection until 15 seconds after they've made their last comment, all comments made within that period would not be saved.
    *   If they then closed/navigated away from the site without making further changes to their comments, the comments would be lost.
* -22 (Thursday) 18:44 MP add that as soon as an edit is made we create a promise on a (30 not 15, it turns out) timeout, and any future edits cancel out the old one and make a new one however, we do also have a visibilitychange listener that fires immediately if you eg tab away. There is no re-fire on hgttp error or timeout, which there would ideally be"
* -23 (Friday) approx 14:30 AJ suggests automated browser testing for annotations is partially repurposed to attempt to record a high number of comments on the platform to recreate error. AL & MP suggest reviewing access logs. 
* -23 (Friday) 16:12 AL confirms access logs for annotations API checked and reports on POST requests, GET requests, responses and server errors.
* -23 (Friday) 16:57 User loses comments a third time. HM updates second report and provides records of discussion with user. 
* -24 (Saturday) 14:19 RT QA is able to manually reproduce error in Google Chrome with varying number of limts on comments (35 - 45) depending on the map and comment length - AL is able to establish that limit is 64kb. Error is not reproducable in Firefox. Error is reproducable in Safari.
* -26 (Monday) 10:29 MP is able to implement fix to staging
* -27 (Tuesday) 13:09 Fix v154 is deployed to production. Some limitations remain.
* -29 (Thursday) 15:44 Fix v155 is deployed to production. Some limitations remain - Google Chrome is only saving every 30 seconds, and closing the broswer tab during the save can cause comments to be lost. 