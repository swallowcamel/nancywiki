## Setting up the development environment
We have been using Mono 2.10.2, anything earlier produces build errors. The version of MonoDevelop that we have been using is the 2.6 Beta 3 build. The instructions are the same for both Linux and OS X installations.

## Third-party dependencies
NDjango is built using F#, so in order to compile and execute the NDjango engine you will need to have F# and the [FSharp PowerPack](http://fsharppowerpack.codeplex.com) installed. Starting with Mono 2.10, F# comes bundled with the installation, so what is left to do is [download the latest version](http://fsharppowerpack.codeplex.com/releases/view/45593#DownloadId=122711) of the FSharp PowerPack and get it installed.

_Note: If you are installing Mono on Linux you need to install the mono-complete package, or you will not get the F# bundle. If you install the mono-core package you will also have to install the fsharp package yourself._

Once downloaded you need to unzip and install it into the GAC

	sudo unzip FSharpPowerPack.zip -d /opt
	cd /opt/FSharpPowerPack-2.0.0.0/bin
	sudo gacutil /i FSharp.Compiler.CodeDom.dll
	sudo gacutil /i FSharp.PowerPack.Build.Tasks.dll
	sudo gacutil /i FSharp.PowerPack.Compatibility.dll
	sudo gacutil /i FSharp.PowerPack.dll
	sudo gacutil /i FSharp.PowerPack.Linq.dll
	sudo gacutil /i FSharp.PowerPack.Metadata.dll
	sudo gacutil /i FSharp.PowerPack.Parallel.Seq.dll

To verify that the installation was successful run the following commands

	$ fsi
	
	Microsoft (R) F# 2.0 Interactive build (private)
	Copyright (c) 2002-2010 Microsoft Corporation. All Rights Reserved.
	
	For help type #help;;
	
	> #r "FSharp.PowerPack.dll";;
	
	--> Referenced '/opt/FSharpPowerPack-2.0.0.0/bin/FSharp.PowerPack.dll'

    > #quit;;

## Build configurations
The _Nancy.sln_ file contains two, Mono specific, build configurations, called _MonoDebug_ and _MonoRelease_, that should be build when building Nancy on Mono. These ensures that windows-only features (such as the WCF host) will not produce build errors when you compile the source.

## Debugging xUnit Tests Using MonoDevelop
If you ever find yourself in a position where you need to debug one of the xUnit tests in Nancy on Mono you should read 
[Debugging xUnit Tests Using MonoDevelop](http://www.grumpydev.com/2011/06/30/debugging-xunit-tests-using-monodevelop) by our very own [Steven Robbins](http://twitter.com/#!/Grumpydev).