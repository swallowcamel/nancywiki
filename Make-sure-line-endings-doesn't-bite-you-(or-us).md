## All good things must come to an end.. 

Before you begin working with Nancy, be sure to set your AutoCRLF option to **_false_**. This tells your git client how it should treat line-endings in your local copy of a repository and prevents you from ending up with Git informing you that every single file have changes in it, when you are pretty damn sure you never touched any of them. Do yourself (and us - we don't want your screwed up line ending commits if we can avoid it!) a favor and make sure this is configured correctly.

To set AutoCRLF for just the Nancy repository, make sure you are in the correct folder and execute

`git config core.autocrlf false`

To make it the default setting for all repositories execute (this might very well introduce the issue in other repositories, if they rely on another setting)

`git config --global core.autocrlf false`
