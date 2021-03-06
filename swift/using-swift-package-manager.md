# Using Swift Package Manager

**Xcode 8 and Swift 3 is soooo coool!**

Because of the addition of the Swift Package Manager to the `swift`.

You can use the Swift Package Manager to

- Create a package
- Build and test
- Generate xcode project file
- Run tests with code coverage reports (kind of)

## Commonly Used Commands


### Create a Package with Swift Package Manager

```sh
# Create
> mkdir /path/to/project
> cd /path/to/project
> swift package init
```

### Build and Test with `swift`

```sh
# Build
> swift build

# Test
> swift test
```

### Test with Code Coverage Enabled

Workaround for `swift` and codecov do not work together as of now.

```sh
# Generate Xcode project with code coverage enabled
> swift package generate-xcodeproj --enable-code-coverage

# Run tests with code coverage
> xcodebuild -scheme <Scheme> -enableCodeCoverage YES test
```

## Example CI Configuration


```yml
language: objective-c
osx_image: xcode8
script:
  - set -o pipefail
  - xcodebuild -version
  - xcodebuild -showsdks
  # - swift test
  - swift package generate-xcodeproj --enable-code-coverage
  - xcodebuild -scheme ExpenseKit -enableCodeCoverage YES test | xcpretty
after_success:
  - bash <(curl -s https://codecov.io/bash)
```

https://github.com/NicholasTD07/ExpenseKit/blob/master/.travis.yml

## Example Project

I put this ExpenseKit together in one night with the help of swift compiler, `swift`. Although ExpenseKit's functionality is very basic and limited, it's very easy to set up a project with Swift Package Manager and get it building and testing with code coverage report on CI. Good user experience in general :)

[ExpenseKit: Protocol Oriented Expanse and Budget Managing Kit in Swift 3](https://github.com/NicholasTD07/ExpenseKit)

### Caveats 

- The generated project only contains a macOS target as of now...
