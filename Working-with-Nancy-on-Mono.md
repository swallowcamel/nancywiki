## We're getting there

We have a goal and that is to make Nancy run as smoothly as possible on Mono and to be able to use MonoDevelop. There are a couple of other things we want to get out of the way first, but we have started to iron out the existing issues with mono compatibility. Right now we are recommending you use **Mono 2.10** since it has improved support for the _dynamic_ keyword, something we make extensive use of.

So a couple of quick facts about Nancy on mono, or as I like to call the branch; Moncy!

* We're very much at an alpha stage with Nancy on mono - _keep that in mind while you read this list_
* There are separate build configurations for mono, not all projects in the solution will compile under mono (eg WCF Hosting), so just build the ones you need.
* We currently do not build the mono version with our build script
* We've successfully ran Nancy on Mono 2.10 on both Linux and Mac OSX
* You'll need F# installed to use the NDjango view engine
* Neither of the view engines are really that well tested on mono yet
* We've not had time to try out the various bootstrappers on Nancy, though the integrated on ([TinyIoC](https://github.com/grumpydev/TinyIoC)) runs like a champ on mono
* You can use our (Google User Group)[https://groups.google.com/forum/#!forum/nancy-web-framework] to post questions about Nancy and Mono
* We've not gotten around to setting up the different test runners on mono yet, so tests are excluded right now
* We could (nearly) **desperatly** do with a couple of more contributors that would like to help out with this!