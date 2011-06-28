* Mono 2.10.2
* MonoDevelop 2.6 Beta 3
* Install FSharp package
* Download the zip of the latest version of [FSharp PowerPack](http://fsharppowerpack.codeplex.com/releases/view/45593#DownloadId=122711)
* Install FSharp PowerPack into the GAC

    $ sudo unzip FSharpPowerPack.zip -d /opt
    $ cd /opt/FSharpPowerPack-2.0.0.0/bin
    $ sudo gacutil /i FSharp.Compiler.CodeDom.dll
    $ sudo gacutil /i FSharp.PowerPack.Build.Tasks.dll
    $ sudo gacutil /i FSharp.PowerPack.Compatibility.dll
    $ sudo gacutil /i FSharp.PowerPack.dll
    $ sudo gacutil /i FSharp.PowerPack.Linq.dll
    $ sudo gacutil /i FSharp.PowerPack.Metadata.dll
    $ sudo gacutil /i FSharp.PowerPack.Parallel.Seq.dll

* Verify that the installation was correct

$ fsi

Microsoft (R) F# 2.0 Interactive build (private)
Copyright (c) 2002-2010 Microsoft Corporation. All Rights Reserved.

For help type #help;;

> #r "FSharp.PowerPack.dll";;

--> Referenced '/opt/FSharpPowerPack-2.0.0.0/bin/FSharp.PowerPack.dll'