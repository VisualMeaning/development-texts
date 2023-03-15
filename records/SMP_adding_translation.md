# Process of adding a new string using the Translation component in sm_platform
When changes are made with a string that uses the Translate component or translate() function in a PR, the PR should ensure that the correspoding Chinese and Russian language files are updated as well.
The steps are as follows:
* Run `npm run extract` to make `en.json` file updated
* Manully updates `ru.json` file, ask Sasha for language help.
* Manully updates `zh.json` file, ask Mingya for language help.
