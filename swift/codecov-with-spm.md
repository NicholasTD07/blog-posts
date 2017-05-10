# Generate code coverage report with Swift Package Manager

I really enjoy TDD a small framework with the Swift Package Manager.

- Simple setup
- Minimum effort to re-run the tests. Just one line of `swift test`.
- Best of all: no need for Xcode

While I can easily do TDD with the SPM, one thing I missed a lot that Xcode offers is the code coverage report.

I found out a way to get nice code coverage reports without Xcode!

### Set up

```sh
gem install xcov # required | might need sudo
gem install fastlane # optional | might also need sudo
```

### Generate code coverage report

```sh
# Need a xcodeproj file first
swift package generate-xcodeproj

# Run the test with either `xcodebuild`
xcodebuild test -scheme TDRedux -enableCodeCoverage YES
# OR fastlane
fastlane scan --code_coverage

# Generate report with `xcov` gem
xcov
```
