## Hey! Watch were you put that thing

If you make use of 3rd party dependencies, in your contributions, make sure they are places in the `/dependencies/name` folder. Do not include any version numbers in the folder name for the dependencies, it doesn't make any sense and it's just going to be a nightmare to maintain. Also make sure you include all needed dependencies, there is nothing (ok, maybe a slight exaggeration) worse than having to be forced to chase around all over the web for downloads, just to be able to build source code you downloaded from some random project. Nancy should built, out of the box, once you've pulled down the source code.

## Make sure you've read the Yoda gospels

Not only should all needed dependencies be included in the dependency folder, but _make sure you read the license_ for the dependency you are about to include in Nancy. Make sure you are not violating anything in it! There are plenty of [sites](http://www.opensource.org/licenses/alphabetical) that explain the hoard of different licenses out there.

We'd rather avoid getting a nice letter from a man in a suite because we are shipping their library x in a way that is not conforming to their choice of license. Nancy is distributed under an [MIT license](http://www.opensource.org/licenses/mit-license.php) which makes it _very_ open to how it is used, not everyone thinks their stuff should be used in such a liberal way and might not be compatible with MIT.



