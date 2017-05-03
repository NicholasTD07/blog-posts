# Fix and issues with updating firebase's subspec

In our iOS project, we are using CocoaPods to include Firebase's RemoteConfig SDK, like this:

```ruby
# the only pod we have from Firebase
pod "Firebase/RemoteConfig", "3.6.0"
```

The latest version of `Firebase/RemoteConfig` is 3.17.0 and it also comes with some fixes for crashes caused by the SDK itself which we have seen in Fabric's crash reports.

## HOW HARD CAN IT BE?

1. Update the version of the pod in `Podfile`
2. `pod update Firebase/RemoteConfig`
3. **PROFIT!** Right?

Yeah... I tried that and CocoaPods gave me this...

```
[!] Unable to satisfy the following requirements:

- `FirebaseAnalytics (= 3.4.2)` required by `Podfile.lock`
- `FirebaseAnalytics (= 3.9.0)` required by `Firebase/Core (3.17.0)`

Specs satisfying the `FirebaseAnalytics (= 3.4.2), FirebaseAnalytics (= 3.9.0)` dependency were found,
 but they required a higher minimum deployment target.
```

## What if...

Yeah... I also tried `pod update Firebase/RemoteConfig FirebaseAnalytics`. It did "work" as in resolving and updating the pods.

However, as soon as I tried to build the app, Xcode fails when during the linker stage with errors like this:

```
duplicate symbol _OBJC_CLASS_$_FIRBundleUtil in: ...
duplicate symbol _OBJC_METACLASS_$_FIRBundleUtil in: ...

ld: 126 duplicate symbols for architecture arm64
```

## THE FIX and why it works!

### FIX:

```sh
# basicall asking CocoaPod to update all pods that are introduced by `pod 'Firebase/RemoteConfig'
$ pod update Firebase FirebaseAnalytics Firebase/Core FirebaseInstanceID FirebaseRemoteConfig
```

### Why didn't previous attempts work?

There was an internal structrue change for the Firebase pods (without a major version change). Put simply, things were moved around within the Firebase pod. New versions of the pods **should not** be used with old versions of the pods. Due to the previous command only updated some (not all) the Firebase pods in our project, there are files that should only be included once were included twice.

### Why dose the FIX work?

It updates all the pods coming from Firebase to the latest versions.

## How can this be prevented?

In my opinion, it might be prevented if the Firebase team updated the dependencies' versions in the podspecs accordingly. But I didn't look into the case so I don't know what exactly went wrong.

## PS: Why not do `pod update`?

We prefer only to update our pods only when it is **neccesary** and We pinned all the pods to specific versions.
