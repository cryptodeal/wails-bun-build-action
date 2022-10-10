# cryptodeal/wails-bun-build-action@v0.0.1

GitHub action to build Wails.io, the action will install GoLang, Bun.sh Runtime, and run `wails build`.
this is to be used on a [Wails.io](https://wails.io) v2 project.

By default, the action will build and upload the results to github, on a tagged build it will also upload to the release.

# Default build

```yml
- uses: cryptodeal/wails-bun-build-action@v0.0.1
  with:
    build-name: wailsApp
    build-platform: linux/amd64
```

## Build with No uploading

```yml
- uses: cryptodeal/wails-bun-build-action@v0.0.1
  with:
    build-name: wailsApp
    build-platform: linux/amd64
    package: false
```

## GitHub Action Options

| Name                                 | Default              | Description                                        |
| ------------------------------------ | -------------------- | -------------------------------------------------- |
| `build-name`                         | none, required input | The name of the binary                             |
| `build`                              | `true`               | Runs `wails build` on your source                  |
| `sign`                               | `false`              | After build, signs and creates signed installers   |
| `package`                            | `true`               | Upload workflow artifacts & publish release on tag |
| `build-platform`                     | `darwin/universal`   | Platform to build for                              |
| `wails-version`                      | `latest`             | Wails version to use                               |
| `wails-build-webview2`               | `download`           | Webview2 installing [download,embed,browser,error] |
| `go-version`                         | `1.17`               | Version of Go to use                               |
| `sign-macos-app-id`                  | ''                   | ID of the app signing cert                         |
| `sign-macos-apple-password`          | ''                   | MacOS Apple password                               |
| `sign-macos-app-cert`                | ''                   | MacOS Application Certificate                      |
| `sign-macos-app-cert-password`       | ''                   | MacOS Application Certificate Password             |
| `sign-macos-installer-id`            | ''                   | MacOS Installer Certificate id                     |
| `sign-macos-installer-cert`          | ''                   | MacOS Installer Certificate                        |
| `sign-macos-installer-cert-password` | ''                   | MacOS Installer Certificate Password               |
| `sign-windows-cert`                  | ''                   | Windows Signing Certificate                        |
| `sign-windows-cert-passowrd`         | ''                   | Windows Signing Certificate Password               |

## Example Build

```yml
name: Wails build

on: [push, pull_request]

jobs:
  build:
    strategy:
      fail-fast: false
      matrix:
        build:
          [
            { name: wailsTest, platform: linux/amd64, os: ubuntu-latest },
            { name: wailsTest, platform: windows/amd64, os: windows-latest },
            { name: wailsTest, platform: darwin/universal, os: macos-latest },
          ]
    runs-on: ${{ matrix.build.os }}
    steps:
      - uses: actions/checkout@v2
        with:
          submodules: recursive
      - uses: cryptodeal/wails-bun-build-action@v0.0.1
        with:
          build-name: ${{ matrix.build.name }}
          build-platform: ${{ matrix.build.platform }}
```

## MacOS Code Signing

You need to make two gon configuration files, this is because we need to sign and notarize the .app before making an installer with it.

```yml
- uses: cryptodeal/wails-bun-build-action@v0.0.1
  with:
    build-name: wailsApp
    sign: true
    build-platform: darwin/universal
    sign-macos-apple-password: ${{ secrets.APPLE_PASSWORD }}
    sign-macos-app-id: ${{ secrets.MACOS_DEVELOPER_CERT_ID }}
    sign-macos-app-cert: ${{ secrets.MACOS_DEVELOPER_CERT }}
    sign-macos-app-cert-password: ${{ secrets.MACOS_DEVELOPER_CERT_PASSWORD }}
    sign-macos-installer-id: ${{ secrets.MACOS_INSTALLER_CERT_ID }}
    sign-macos-installer-cert: ${{ secrets.MACOS_INSTALLER_CERT }}
    sign-macos-installer-cert-password: ${{ secrets.MACOS_INSTALLER_CERT_PASSWORD }}
```

`build/darwin/gon-sign.json`

```json
{
  "source": ["./build/bin/wailsApp.app"],
  "bundle_id": "com.wails.app",
  "apple_id": {
    "username": "username",
    "password": "@env:APPLE_PASSWORD"
  },
  "sign": {
    "application_identity": "Developer ID Application: XXXXXXXX (XXXXXX)",
    "entitlements_file": "./build/darwin/entitlements.plist"
  },
  "dmg": {
    "output_path": "./build/bin/wailsApp.dmg",
    "volume_name": "Lethean"
  }
}
```

`build/darwin/gon-notarize.json`

```json
{
  "notarize": [
    {
      "path": "./build/bin/wailsApp.pkg",
      "bundle_id": "com.wails.app",
      "staple": true
    },
    {
      "path": "./build/bin/wailsApp.app.zip",
      "bundle_id": "com.wails.app",
      "staple": false
    }
  ],
  "apple_id": {
    "username": "USER name",
    "password": "@env:APPLE_PASSWORD"
  }
}
```

`build/darwin/entitlements.plist`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>com.apple.security.app-sandbox</key>
  <true/>
  <key>com.apple.security.network.client</key>
  <true/>
  <key>com.apple.security.network.server</key>
  <true/>
  <key>com.apple.security.files.user-selected.read-write</key>
  <true/>
</dict>
</plist>
```
