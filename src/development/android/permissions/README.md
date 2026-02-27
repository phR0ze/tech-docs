# Permissions

### Quick links
* [Storage Access](#storage-access)
  * [Scoped Storage](#scoped-storage)
  * [Shared Storage](#shared-storage)

# Storage Access
Android is an old platform that has been patched and retrofitted many times to deprecate APIs and 
boltg on new security patterns and as such it is quite confusing to navigate. Additionally there is a 
lot of documentations and guides that attempt to work around the new security measures by targetting 
older APIs etc... In the end if you want to publish your app in the play store you need to make sure 
it is using modern APIs and following the latest guidance.

`External storage` aka `shared storage` are the terms Android uses to describe the user space storage 
i.e. storage for the user or the user's apps to store files. They use the term `internal storage` to 
describe space the system uses for its files. However Android has many security mechanisms 
controlling how the external or shared storage can be accessed.

If your app only needs its own files and settings then `Scoped Storage` is fine and you don't need to 
worry about permissions or anything else. If however, your app needs to interact with the rest of the 
system or read or write to files it didn't create then `Scoped Storage` aka `app only storage` is not 
enough and you need to look into `Shared Storage` which is way more complicated.

***NOTE: requires Android 10 (API level 29) or higher for these instructions to work correctly***

## Scoped Storage
Modern Android apps i.e. targeting Android 10 (API level 29) or higher are given scoped access to 
external storage by default aka `Scoped Storage`. This restricts the app's access to the app-specific 
directory on external storage or specific types of media the app creates. There are no permissions to 
worry about or extra steps, it just works great for files you app creates e.g. preferences and 
internal files. For each app, the system provides directories within internal storage and external 
storage where an app can organize its files. One directory is designed for persistent files and 
another for cached files. 

For this use case an app would simply get the provided storage paths from the system and use the 
`File` API directly to access and store files as needed to theses app specific provided paths.

## Shared Storage
Shared storage resides on the external storage device and is wrapped in a number of security 
constraints. You can access it in three different ways:
1. `MediaStore`
  * provides access to media in well known public directories
  * does ***NOT*** provide access to other files types such as PDFs
2. `Storage Access Framework` provides access to other file types such as PDFs, epubs etc...
3. `BlobStoreManager` provides access to large datasets like machine learning or media playback

### Legacy External Storage
It is possible to opt out of the new scoped storage by targeting the older Android 10 (API level 29) 
SDK as your target using the [requestLegacyExternalStorage attribute](https://developer.android.com/about/versions/11/privacy/storage).
This flag allows apps to temporarily opt out of scoped storage and access directories and files 
normally. After you update your app to target Android 11 or higher the system will ignore this 
attribute.

```xml
<application
  android:requestLegacyExternalStorage="true"> 
</application>
```

### Deprecated

**READ_EXTERNAL_STORAGE** - deprecated as of Android 13 (API level 33)
```xml
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"
  android:maxSdkVersion="32" />
```

**WRITE_EXTERNAL_STORAGE** - deprecated as of Android 10 (API level 29)
```xml
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"
  android:maxSdkVersion="29" />
```

### MediaStore
The Android [MediaStore](https://developer.android.com/training/data-storage/shared/media) 
automatically scans all external storage and adds any files it finds to well-defined collections 
separated by MIME type and controlled by specific permissions:

* `MediaStore.Images` - includes image types found in `DCIM/` and `Pictures/`
  * `READ_MEDIA_IMAGES` permission only required for files your app didn't create

* `MediaStore.Video` - includes video types found in `DCIM/`, `Movies`, and `Pictures/`
  * `READ_MEDIA_VIDEO` permission only required for files your app didn't create
  * Note: requesting both IMAGES and VIDEO permissions at the same time will only show a single 
    dialog with both included in it.

* `MediaStore.Audio` - includes audio types found in `Alarms/`, `Audiobooks/`, `Music/`, 
  `Notifications/`, `Podcasts/`, and `Ringtones/`
  * `READ_MEDIA_AUDIO` permission only required for files your app didn't create

* `MediaStore.Downloads` - includes media files in the `Download/` directory similarly to the `Files`
  endpoint below but won't allow for accessing files your app didn't create
  * Use the `Storage Access Framework` to access files you didn't create in this directory

* `MediaStore.Files` - simply allows for viewing the Images, Video or Audio files at the same time 
   rather than calling each colletion separately, but you still need the same permissions.

### Storage Access Framework
Documents and other files such as PDFs or EPUB formats etc... can be accessed using the Storage 
Access Framework.


### Manage External Storage permission
As of `Android 11 (API 30)` [Google Play restricts](https://developer.android.com/training/data-storage/manage-all-files#all-files-access-google-play) the use of `All files access` permission `MANAGE_EXTERNAL_STORAGE` is prohibited.
If you app does not require acces to the `MANAGE_EXTERNAL_STORAGE` permission, you must remove it 
from your app's manifest in order to successfully publish your app. Only apps in categories that 
follow will be allowed to use this permission:
* File managers
* Backup and restore apps
* Anti-virsus apps
* Document management apps
* On-device files search
* Disk and file encryption
* Device to device data migration


Android 10 (API level 29) and higher, apps run in a storage sandbox by default. This sandbox prevents 
your app from accessing files outside of the app-specific directory and publicly-shared directories.



Android 11 (API level 30) and higher has some changes:
* `requestLegacyExternalStorage` is ignored when run on Android 11 or higher devices
* User controlled access via `ACTION_OPEN_DOUCMENT_TREE`
  * no longer allows choosing the root dirctory 
  * no longer allows choosing `Download`
  * no longer allows choosing anything in `Android/data` or `Android/obb`
  * filters returned files from directories by MIME types app has permission to access





A directory is considered a legacy storage location if it isn't an app-specific directory or a 
public shared directory e.g. uses files in `/sdcard`. If you app creates or consumes files in a 
legacy storage location, Android recommends that you migrate your app's files to locations that are 
accessible with scoped storage and make any necessary app changes to work with files in scoped 
storage.

Android continues to add more and more complex file system permissions. IN Android 11 
`MANAGE_EXTERNAL_STORAGE` aka all file access was changed to not allow access to `/Android/data` or 
`Android/obb` without using the SAF (Storage Access Framework). Then in Android 13 this path was 
changed so that only sub directories in these 

The recommended app behavior while using scoped storage:
* app storage its files in `getExternalFilesDir` or `getFilesDir` which gives an absolute path to a 
directory in the shared external storage where your app can place files it owns. These files are 
internal to the app and not typically visible to the user as media.
* any shared non-media files should be in an app created subdirectory in `Downloads`
* use MediaStore's downloads or document collections to export non-media files

Storage Access Framework  
* Grant access to a directory's contents

ContentResolver#loadThumbnail

https://pub.dev/packages/media_store_plus


* Read shared media and documents?

* [MediaStore](https://developer.android.com/reference/android/provider/MediaStore)
* [File manager permissions](https://www.reddit.com/r/Android/comments/wsnhct/file_manager_dev_discovers_a_new_loophole_to_get/)
* [Android storage permissions](https://developer.android.com/training/data-storage/shared/media#storage-permission)
* [Flutter storage permissions](https://stackoverflow.com/questions/75298807/storage-permission-in-android-13-flutter)































### Legacy External Storage

### Opt out of Android 11 API 30 scoped storage access
You can opt out of the API 30 all files access restrictions by targetting Android 10 API 29 or lower 
and setting the `requestLegacyExternalStorage="true"` flag.

Note: `targetSdk` is the highest SDK version your app is known to work with while `compileSdk` is the 
version your compiling with which doesn't have a runtime impact only compile time checks.

1. Edit `app/manifests/AndroiManifest.xml` and add
```xml
<manifest ... >
  <application
    android:requestLegacyExternalStorage="true"
    ...
  </application>
</manifest>
```

2. Edit `build.properties` and set `minSdk` to `24` for Android 7 minimum
3. Edit `build.properties` and set `targetSdk` to `29` for Android 10 maximum
4. Click `Sync Now` in the top right section

### Request storage permissions
[Request app permissions](https://developer.android.com/training/permissions/requesting)
Since API 23 you have to request permission at runtime to get access to device capabilities and have 
remained similar enough to use the same approach up until Android 10 API 29 if you use the opt out 
flag.

* [Declare hardware as optional](https://developer.android.com/training/permissions/declaring#hardware-optional)
* [Determine hardware availability](https://developer.android.com/training/permissions/declaring#determine-hardware-availability)

1. In `app/manifests/AndroidManifest.xml` declare the permissions needed
```XML
<manifest ... >
   <uses-permission-sdk-23 android:name="android.permission.CAMERA"/>
   <uses-permission-sdk-23
     android:name="android.permission.READ_EXTERNAL_STORAGE"
     android:maxSdkVersion="29" />
   <uses-permission-sdk-23
     android:name="android.permission.WRITE_EXTERNAL_STORAGE"
     android:maxSdkVersion="29" />
  <application
  ...
  </application>
  <uses-feature android:name="android.hardware.camera" android:required="false"/>
</manifest>
```
2. Navigate to `File >Project Structure... >Dependencies >app` and add the following dependencies
   * `androidx.activity` version `1.2.0` or later
   * `androidx.fragment` version `1.3.0` or later

## Storage Access Framework
Because of the security restrictions in Android 10 and higher we can no longer depend on standard 
storage permissions to grant access to all file types i.e text files are notably excluded. In order 
to gain access to all file types including text files we need to make use of the `Storage Access 
Framework` to request a directory from the user that we can then fully manage. The directory the
user chooses needs to be something other than `root` and the `Download` directory.

### Grant access to a directory's contents
The Android team determined that they could improve security by making an application ask the user 
which directory it can manage using the `ACTION_OPEN_DOCUMENT_TREE` intent action introduced in `API 21`.
In API 30 further restrictions were made to exclude
* `root`
* `Download`
* Anything in `Android/data`
* Anything in `Android/obb`

WARNING: accessing many files through the document tree is known to be slow

