# [WIP] Swift as Tool Chain and CI

**Xcode 8 and Swift 3**

## Commonly Used Commands

```sh
# Create
> mkdir /path/to/project
> cd /path/to/project
> swift package init --type library

# Build
> swift build

# Test
> swift test

# Generate Xcode project with code coverage enabled
> swift package generate-xcodeproj --enable-code-coverage

# Run tests with code coverage
> xcodebuild -scheme <Scheme> -enableCodeCoverage YES test
```

## Example CI Configuration

`swift test` and CodeCov do not work very well as of now.

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
