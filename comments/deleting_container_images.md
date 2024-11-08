# Instructions for removing images from Azure Container Registries

Deploying new code with a container workflow is (mostly) straight forward - push a new image.
It's less clear what to do with removing images!

Each deployment will create a new image which will have the latest security updates from the upstream Ubuntu repository as they are published.
We do not attempt to manage packages on or update running containers, as deploying with a new image is cleaner and more robust, while also giving us the ability to do simple rollbacks in the case of errors.
Old images will not be used or run, except in this rollback scenario.

## Defender recommendations

Microsoft Defender is an attempt to automate elements of good security practices by analysing config and content for common problems.
In practice it makes a lot of busy-work that has little impact on the real security or robustness of a system as the detected problems do not have enough context to determine whether any are exploitable.

[Microsoft Defender for Cloud: Recommendations] shows us a "Severity High" rule that "Container registry images should have vulnerability findings resolved".
These are images that are present in the registry but have been superseded and are no longer being run.
No other applications have access to the registry to deploy images.

Further, the mechanism used to tell if an image has a vulnerability is comparing package versions in the image against a CVE database.
[The bogus CVE problem] and the lack of context on how those packages are used in the wider system means essentially all reports are valueless.

It would be a poor use of time to mark individual CVEs and images from the automated reports as not being affected. Time spent on, for example, code review is much more impactful to the overall security of the system.

Being able to perform a simple rollback helps both robustness and security of the system, minimising downtime or window for exploitation in the event of an incident.

Removing old images ensures the latest updates for packages are the only ones that can be deployed, but that may be an overstated risk compared to the ability to easily revert to a previous known state of working packages.

On balance, we'd probably prefer to keep at least the last few images used around, however provided each update is performed and passes tests, removing all but the currently running image avoids having to explain all this, with little added risk.


## Untagging does not delete

The final wrinkle is to [delete container images in Azure Container Registry] it's not sufficient to just use docker commands. That can untag an image, but will not trigger a clean up of layers, so the vulnerable image may still get reported.
Instead, images need to be removed by digest using a specific Azure command.


## Cleaning instructions

Script below assumes `az`, `docker`, and `jq` are installed.

```sh
export SUBSCRIPTION=617e5812-8cfb-4720-a658-e4c7565df9c7
export TENANT=29509fb2-7faf-4f8b-b7a2-32f96ec5de6c
export ACR_REGISTRY_NAME=heauksdsdevsmpcr
export TAG=smp

az login --tenant ${TENANT}
az acr login --name ${ACR_REGISTRY_NAME} --subscription ${SUBSCRIPTION}

az acr manifest list-metadata --name ${TAG} --registry ${ACR_REGISTRY_NAME} |
  jq '.[] | select(any(.tags[]?; . == "latest") | not) | "'${TAG}'@" +  .digest' |
  xargs --no-run-if-empty -L1 \
    az acr repository delete --name ${ACR_REGISTRY_NAME} --yes --image
```

All images except those tagged 'latest' will be removed by this operation, and the registry name should be changed to match the dev or prod service as desired.



[Microsoft Defender for Cloud: Recommendations]: https://portal.azure.com/#view/Microsoft_Azure_Security/RecommendationsBladeV2
[The bogus CVE problem]: https://lwn.net/Articles/944209/
[delete container images in Azure Container Registry]: https://learn.microsoft.com/azure/container-registry/container-registry-delete

