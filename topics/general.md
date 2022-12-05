# Bluace development guidelines for Business Central

## General

### Required tools
At Bluace we develop AL code in a decentralized way. This means that each developer creates/modifies AL apps based on a docker system on the local laptop/system and check the code modification in into our source code manamement system, Git.

The Business Central development docker containers are created by BC Container Helper, a powershell module that is released by Microsoft to assist us with automating recurring development tasks. We use a development container as short as possible, to avoid that we miss changes that are done by Microsoft in the base applications.

This way of working leads to the following tools that must be installed on a developer system:

* [Git](https://git-scm.com/)
* [BC Container Helper](https://www.powershellgallery.com/packages/BcContainerHelper/)
* [Docker Desktop](https://www.docker.com/)
* [Visual Studio Code](https://www.vscode.com/)

Please note that Docker Desktop must be setup to use Windows Containers to be able to use it for Business Central development. To be able to do that, the following Windows features must be installed:

![Containers and Hyper-V Windows features must be installed](../images/general/required-tools-windows-features.png)

### Required Visual Studio Code extensions

To be able to work with the AL language, the AL Language extension by Microsoft must be installed into Visual Studio Code. 
* [AL Language](https://marketplace.visualstudio.com/items?itemName=ms-dynamics-smb.al)
* [waldo's CRS AL Language Extension](https://marketplace.visualstudio.com/items?itemName=waldo.crs-al-language-extension)
* [XLIFF Sync](https://marketplace.visualstudio.com/items?itemName=rvanbekkum.xliff-sync)
* [AZ AL Dev Tools/AL Code Outline](https://marketplace.visualstudio.com/items?itemName=andrzejzwierzchowski.al-code-outline)

### Recommended Visual Studio Code extensions
* [AL Extension Pack](https://marketplace.visualstudio.com/items?itemName=waldo.al-extension-pack)

### Recommended tools
* [Git Extensions](https://gitextensions.github.io/)
* [Postman](https://www.postman.com/)

[:arrow_left:](../README.md) [Back](../README.md)