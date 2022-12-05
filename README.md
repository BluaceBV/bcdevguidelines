# Bluace development guidelines for Business Central

The goal of this document is to give the developer some guidence on the processes that we use for AL development at Bluace. That is the process from issue to realized product. It covers the registration of issues, creating tasks, developing the AL code on a task in a feature branche and finally pull requesting the modified feature branch into the development branch.

## Index

### [Azure DevOps](topics/azure-devops.md)

* [User Stories](topics/azure-devops.md#user-stories)
* [Tasks](topics/azure-devops.md#tasks)
* [Rework task (dev/test)](topics/azure-devops.md#rework-task-devtest)
* [Repository/Branches](topics/azure-devops.md#repositorybranches)
* [Pull Requests](topics/azure-devops.md#pull-requests)
* [Use PowerBI dashboard](topics/azure-devops.md#use-powerbi-dashboard)

### [BC Container Helper](topics/bc-container-helper.md)

* [Creating local development environment](topics/bc-container-helper.md#creating-local-development-environment)
* [Example script](topics/bc-container-helper.md#example-script)

### [AL Development](topics/al-development.md)

* [Recommended tools/apps](topics/al-development.md#recommended-toolsapps)
* [Using the development branch](topics/al-development.md#using-the-development-branch)
* [Object centric programming](topics/al-development.md#object-centric-programming)
* [Object numbering](topics/al-development.md#object-numbering)
* [Naming](topics/al-development.md#naming)
* [Use method codeunits for actions](topics/al-development.md#use-method-codeunits-for-actions)
* [Creating event publishers](topics/al-development.md#creating-event-publishers)
* [Creating event subscribers](topics/al-development.md#creating-event-subscribers)
* [Using event subscribers](topics/al-development.md#using-event-subscribers)
* [RequestPages](topics/al-development.md#requestpages)
* [Promoted actions](topics/al-development.md#promoted-actions)
* [Update permission sets](topics/al-development.md#update-permission-sets)
* [Captions/Tooltips](topics/al-development.md#captionstooltips)
* [Translations using XLIFF Sync](topics/al-development.md#translations-using-xliff-sync)
* [Clean code](topics/al-development.md#clean-code)
* [Using pragma](topics/al-development.md#using-pragma)

## Recommended blogs

* [Erik Wouters](https://www.waldo.be/)
* [Vjekoslav Babić](http://www.vjeko.com/)
* [Arend-Jan Kauffmann](https://www.kauffmann.nl/)
* [Erik Hougaard](https://www.hougaard.com/)
* [Stefano Demiliani](https://www.demiliani.com)
* [Yun Zhu](https://yzhums)