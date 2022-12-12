# Bluace development guidelines for Business Central

The goal of this document is to give the developer some guidence, as part of our toolkit, on the processes that we use for AL development at Bluace. That is the process from issue to realized product. It covers the registration of issues, creating tasks, developing the AL code on a task in a feature branche, documenting the applied changes and finally pull requesting the modified feature branch into the development branch. This document applies to both product development as customer specific development.

Part of our job is also to be informed about the latest developments in the broader ecosystem of the Business Central product. This enables us to make educated choices regarding functionalities that we create and maintain. This information is mainly published through digital channels, on Yammer or through blogs of MVP's. The links to a couple of inportant ones are included in this document.

Because a guideline is never complete, this document is published in a GitHub repository. This enables you to suggest changes as pull requests, so we can collectively work on documenting the way we work on daily basis.

## Index

### [General](topics/general.md)
* [Required tools](topics/general.md#required-tools)
* [Required Visual Studio Code extensions](topics/general.md#required-visual-studio-code-extensions)
* [Recommended Visual Studio Code extensions](topics/general.md#recommended-visual-studio-code-extensions)
* [Recommended tools](topics/general.md#recommended-tools)

### [Azure DevOps](topics/azure-devops.md)

* [Projects](topics/azure-devops.md#projects)
* [Boards](topics/azure-devops.md#boards)
* [Repos](topics/azure-devops.md#repos)
* [Pipelines](pipelines/azure-devops.ms#pipelines)
* [Use PowerBI dashboard](topics/azure-devops.md#use-powerbi-dashboard)

### [BC Container Helper](topics/bc-container-helper.md)

* [Creating local development environment](topics/bc-container-helper.md#creating-local-development-environment)
* [Example script](topics/bc-container-helper.md#example-script)

### [AL Development](topics/al-development.md)

* [Using the development branch](topics/al-development.md#using-the-development-branch)
* [CodeCops](topics/al-development#codecops)
* [Object centric programming](topics/al-development.md#object-centric-programming)
* [Object numbering](topics/al-development.md#object-numbering)
* [Naming](topics/al-development.md#naming)
* [Use method codeunits for actions](topics/al-development.md#use-method-codeunits-for-actions)
* [Creating event publishers](topics/al-development.md#creating-event-publishers)
* [Creating event subscribers](topics/al-development.md#creating-event-subscribers)
* [Using event subscribers](topics/al-development.md#using-event-subscribers)
* [RequestPages](topics/al-development.md#requestpages)
* [Promoted actions](topics/al-development.md#promoted-actions)
* [Using ApplicationArea](topics/al-development.md#using-applicationarea)
* [Update permission sets](topics/al-development.md#update-permission-sets)
* [Captions/Tooltips](topics/al-development.md#captionstooltips)
* [Translations using XLIFF Sync](topics/al-development.md#translations-using-xliff-sync)
* [Remove unused parts](topics/al-development.md#remove-unused-parts)
* [Compiler directives](topics/al-development.md#compiler-directives)
* [Confirm Management](topics/al-development.md#confirm-management)
* [Dialogs](topics/al-development.md#dialogs)

## Recommended links
* [Business Central Yammer (Partners only: access controlled by Microsoft)](https://www.yammer.com/dynamicsnavdev/)
* [Erik Wouters](https://www.waldo.be/)
* [Vjekoslav Babić](http://www.vjeko.com/)
* [Arend-Jan Kauffmann](https://www.kauffmann.nl/)
* [Erik Hougaard](https://www.hougaard.com/)
* [Stefano Demiliani](https://www.demiliani.com)
* [Yun Zhu](https://yzhums)