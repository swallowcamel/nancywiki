If you have problems with building with rake you should make sure of a few things

1. Make sure you have Ruby 1.9.x installed and get Rake and Albacore gems as well
2. If you are having problems with building on windows make sure your CMD codepage is set to UTF-8 using `chcp 65001` the default is usually something else. (note: this is because of a character Andreas' last name ;P)
3. If you want to build your own custom nuget package you can run `rake nuget_package` and it will be in the `/build/nuget` folder
4. Make sure no tests are failing, if a test fails the build is aborted and you really should keep the tests passing ;-)