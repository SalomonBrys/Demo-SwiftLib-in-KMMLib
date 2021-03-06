# Swift static library inside a Kotlin/Native lib

This Kotlin/Multiplatform Mobile demo library project demonstrates how to use a **Swift static library** into a **Kotlin/Native KLib** in order to use it as a dependency in another _Kotlin/Multiplatform Mobile_ project.

This demoes the use of the cryptography algorithm _Chacha20-Poly1305_ that is only available in iOS as a Swift API. 

## Points of interest

- The [SwiftCryptoKit directory](https://github.com/SalomonBrys/Demo-SwiftLib-in-KMMLib/tree/main/SwiftCryptoKit) contains an XCode project that generates a Swift static library:
    - All its Swift classes and methods are `public` and `@objc` annotated to be accessible from Objective-C & Kotlin.
    - An "Objective-C Bridging Header" is configured for Objective-C interoperability.
    - A "Run Script" build phase is added at the end of the build to export the Swift Objective-C header with the library.
    - A Gradle build script was added to enable the root Gradle project to compile the library with `xcodebuild`.
- The root directory contains a Gradle project that provides a Kotlin/Native wrapper around the Swift static library:
    - The [Gradle build script](https://github.com/SalomonBrys/Demo-SwiftLib-in-KMMLib/blob/main/build.gradle.kts) declares a C-Interop that depends on the Swift static library
    - The [interop def file](https://github.com/SalomonBrys/Demo-SwiftLib-in-KMMLib/blob/main/src/nativeInterop/cinterop/SwiftCryptoKit.def) defines the linker flags that needs to be used when linking with the library:
        - **`-ios_simulator_version_min 13.0.0` and `-iphoneos_version_min 13.0.0` are needed because Swift interoperability needs iOS 13.0 or newer**.
        - Swift library search path are also needed.
    - Note that any KMM appplication depending on the library must target iOS 13.0 or newer.
- You can run `./gradlew publishToMavenLocal` and then have a look at the `~/.m2/repository/org/demo/crypto/` directory to see all the files that would be published.

## Why, oh why?

Apple is pushing its Swift language more and more for iOS development.
Some new native features that are added to the system are being made available only in Swift: this is the case for example of the new **CryptoKit** framework, which exposes some crypto algorithm that weren't available before, such as _ChaCha20-Poly1305_.
Therefore, if we want to use a hardware-optimised implementation of such algorithms, we have no choice but use Swift.

While Kotlin/Native does not interface with Swift, both Swift & Kotlin/Native interop really well with Objective-C, so it is rather simple to create a small static library that exposes some Swift-only APIs (such as ChaChaPoly in CryptoKit) to Objective-C and therefore to Kotlin/Native.

This demos a Kotlin/Multiplatform Mobile **library** (KLib) that provides an API for the ChaCha20-Poly1305 algorithm and uses the system's most efficient implementation for each target.
This library would then be used as a dependencies on multiple Kotlin/Multiplatfom Mobile application projects.

As Apple is releasing more and more features in Swift only, this use case is becoming more and more important.
