# Unpub [![build](https://img.shields.io/travis/bytedance/unpub.svg)](https://travis-ci.org/bytedance/unpub) [![pub](https://img.shields.io/pub/v/unpub.svg)](https://pub.dev/packages/unpub)

Unpub is a private Dart Pub server for Enterprise, with a simple web interface to search and view packages information.

![Screenshot](https://raw.githubusercontent.com/bytedance/unpub/master/assets/screenshot.png)

## Usage via command line

```sh
pub global activate unpub
unpub --database mongodb://localhost:27017/dart_pub # Replace this with production database uri
```

Unpub use mongodb as meta information store and file system as package(tarball) store by default.

Dart API is also available for further customization.

## Usage via Dart API

### Example

```dart
import 'package:unpub/unpub.dart' as unpub;

main(List<String> args) async {
  var basedir = '/path/to/basedir'; // Base directory to save pacakges
  var db = 'mongodb://localhost:27017/dart_pub'; // MongoDB uri

  var metaStore = unpub.MongoStore(db);
  await metaStore.db.open();

  var packageStore = unpub.FileStore(basedir);

  var app = unpub.App(
    metaStore: metaStore,
    packageStore: packageStore,
  );

  var server = await app.serve('0.0.0.0', 4000);
  print('Serving at http://${server.address.host}:${server.port}');
}
```

### More options

```dart
var app = unpub.App(
  // Specify upstream url, use https://pub.dev by default
  upstream: 'https://some-faster-mirror-of-pub-dev',

  // Http proxy to call googleapis to get uploader email
  googleapisProxy: 'http://i-can-visit-google'

  // If specified, unpub will use this email as uploader instead of requesting googleapis
  overrideUploaderEmail: 'dart-pub@my-company.com'
);
```

### Package validator

Naming conflicts is a common issue for private registry. A reasonable solution is to add prefix to reduce conflict probability.

With `uploadValidator` you could check if uploaded package is valid.

```dart
var app = unpub.App(
  // ...
  uploadValidator: (Map<String, dynamic> pubspec, String uploaderEmail) {
    // Only allow packages with some specified prefixes to be uploaded
    var prefix = 'my_awesome_prefix_';
    var name = pubspec['name'] as String;
    if (!name.startsWith(prefix)) {
      throw 'Package name should starts with $prefix';
    }

    // Also, you can check if uploader email is valid
    if (!uploaderEmail.endsWith('@your-company.com')) {
      throw 'Uploader email invalid';
    }
  }
);
```

### Customize meta and package Store

Unpub is designed to be extensible. It is quite easy to customize your own meta store and package store.

```dart
class MyAwesomeMetaStore extends unpub.MetaStore {
  // Implement methods of MetaStore abstract class
  // ...
}

class MyAwesomePackageStore extends unpub.PackageStore {
  // ...
}

// Then use it
var app = unpub.App(
  metaStore: MyAwesomeMetaStore(),
  packageStore: MyAwesomePackageStore(),
)
```

## Alternatives

- [pub-dev](https://github.com/dart-lang/pub-dev): Source code of [pub.dev](https://pub.dev), which should be deployed at Google Cloud Platform.
- [pub_server](https://github.com/dart-lang/pub_server): An alpha version of pub server provided by Dart team.

## Credits

Web page styles are mostly imported from https://pub.dev directly.

## License

MIT
