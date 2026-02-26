---
applyTo: "**"
---


# Commands & Scripts

- Build: `make`, Xcode build via project.yml (XcodeGen)
- Lint: SwiftLint (if configured), custom scripts in scripts/
- Test: UnitTests, UITests targets, `xcodebuild test`, fastlane
- Deployment: fastlane (see fastlane/Fastfile), Bitrise CI/CD
- Codegen: Typewriter (typewriter.yml), XcodeGen (project.yml)
- Migration: CoreData migrations, manual scripts in scripts/
- Utilities: Dangerfile.swift (PR checks), Gemfile (Ruby deps)