## Setting up the development environment
We have been using Mono 2.10.2, anything earlier produces build errors. The version of MonoDevelop that we have been using is the 2.6 Beta 3 build. 

## Third-party dependencies
The NDjango engine in Nancy was built with FSharp, so in order to build it you are going to have to install the latest **FSharp package** (using your package manager of choice). Once you have installed it, you will also have to download the zip of the latest version of [FSharp PowerPack](http://fsharppowerpack.codeplex.com/releases/view/45593#DownloadId=122711)

Once downloaded you need to unzip and install it into the GAC

	$ sudo unzip FSharpPowerPack.zip -d /opt
	$ cd /opt/FSharpPowerPack-2.0.0.0/bin
	$ sudo gacutil /i FSharp.Compiler.CodeDom.dll
	$ sudo gacutil /i FSharp.PowerPack.Build.Tasks.dll
	$ sudo gacutil /i FSharp.PowerPack.Compatibility.dll
	$ sudo gacutil /i FSharp.PowerPack.dll
	$ sudo gacutil /i FSharp.PowerPack.Linq.dll
	$ sudo gacutil /i FSharp.PowerPack.Metadata.dll
	$ sudo gacutil /i FSharp.PowerPack.Parallel.Seq.dll

To verify that the installation was successful run the following commands

	$ fsi
	
	Microsoft (R) F# 2.0 Interactive build (private)
	Copyright (c) 2002-2010 Microsoft Corporation. All Rights Reserved.
	
	For help type #help;;
	
	> #r "FSharp.PowerPack.dll";;
	
	--> Referenced '/opt/FSharpPowerPack-2.0.0.0/bin/FSharp.PowerPack.dll'

## Build configurations
The _Nancy.sln_ file contains two, Mono specific, build configurations, called _MonoDebug_ and _MonoRelease_, that should be build when building Nancy on Mono. These ensures that windows-only features (such as the WCF host) will not produce build errors when you compile the source.