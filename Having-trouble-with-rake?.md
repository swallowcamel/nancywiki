If you have problems with building with rake you should make sure of a few things

1. Make sure you have Ruby 1.9.x installed and get Rake and Albacore 1.x gems as well
2. If you are having problems with building on windows make sure your CMD codepage is set to UTF-8 using `chcp 65001` the default is usually something else. (note: this is because of a character Andreas' last name ;P)
- rakefile.rb should start with # encoding: utf-8 (this might be an issue if working with older branches)
3. If you want to build your own custom nuget package you can run `rake nuget_package` and it will be in the `/build/nuget` folder
4. Make sure no tests are failing, if a test fails the build is aborted and you really should keep the tests passing ;-)

Note: The latest 2.x version of Albacore contains many breaking changes and won't work with Nancy's build - make sure you install 1.x by using the command `gem install albacore --version "~> 1.0.rc"`, or you can do `bundle install && bundle exec rake`.