# Adding a new locale to the single-locale build

Android has two different types of builds: a multi-locale build that ships in Google Play Store, and a single-locale build (containing only one locale) that is mostly used for testing.

The list of locales for which single-locale builds are created is stored inside a file named **all-locales** within Mozilla’s code repositories. Since the goal is to create Nightly builds, the file will be in mozilla-central: https://hg.mozilla.org/mozilla-central/file/default/browser/locales/all-locales

## File a bug to add the new locale

You need to file a bug in Firefox for Android::General requesting the new locale (see for example [this bug for Guaraní](https://bugzilla.mozilla.org/show_bug.cgi?id=1271839)). You can use this [bug template] to make things faster (update with the appropriate locale code and language name).

You also want to file a bug to use as tracker ([example for gn](https://bugzilla.mozilla.org/show_bug.cgi?id=1271970)), adding an alias like `fm-l10n-gn` for `gn` and blocking `fm-l10n-tracker`.

## Land initial content in l10n repository

With cross-channel, all builds of Firefox and Firefox Android are generated using strings from the l10n-central repository. If a project started localizing using a BitBucket repository, it’s **fundamental** to populate l10n-central with that content when requesting the official repository creation on `https://hg.mozilla.org/l10n-central/LOCALE_CODE`.

After the first content lands in l10n-central, it’s a good idea to perform some basic checks before enabling the build:
* Check `toolkit/global/intl.properties` ([en-US version](https://hg.mozilla.org/mozilla-central/file/default/toolkit/locales/en-US/chrome/global/intl.properties)) for evident mistakes.
* Check if there’s a `region.properties` file in `mobile/chrome/region.properties`. If needed replace it with the [stock version](../searchplugins/files/mobile_region.properties).

## Set up searchplugins

Check the [Set up searchplugins](../searchplugins/setup_searchplugins.md) document for detailed instructions on how to set up searchplugins for new locales.

## Creating a patch for build configuration

First of all make sure that your environment is [correctly set up](../../tools/mercurial/setting_mercurial_environment.md), and update your local mozilla-unified clone:

```BASH
$ cd ~/mozilla/mercurial/mozilla-unified
$ hg pull -r default -u
$ hg update central
```

The file to modify is in `mobile/android/locales/all-locales`, open it with your text editor of choice.

```BASH
$ atom mobile/android/locales/all-locales
```

And add the new locale to the list. With Atom and the Sort Lines package installed, you can press `F5` to make sure that the list is in **alphabetical order**. Let’s say for example that you need to add `ab-CD` to the list of supported locales.

The second file to modify is `mobile/android/locales/l10n.toml`. This is the beginning of the file:

```
# This Source Code Form is subject to the terms of the Mozilla Public
# License, v. 2.0. If a copy of the MPL was not distributed with this
# file, You can obtain one at http://mozilla.org/MPL/2.0/.

basepath = "../../.."

locales = [
    "an",
    "ar",
    "as",
    "ast",
    "az",
...
```

Identify the `locales` section, and add the new locale code between double quotes, followed by a comma. As before, you can use Atom to make sure the list is in alphabetical order (make sure to select only the lines with actual locale codes before pressing `F5`).

Once this is done, identify the `exclude-multi-locale` section. Since this locale is not shipping on multi-locale builds yet, you should add it there too.

After you’ve finished editing the files, check the status of the repository, and the diff.

```BASH
$ hg status
M mobile/android/locales/all-locales

$ hg diff
--- a/mobile/android/locales/all-locales
+++ b/mobile/android/locales/all-locales
@@ -1,8 +1,9 @@
+ab-CD
 ar
 be
 ca
 cs
 da
 de
 es-AR
 es-ES

 diff --git a/mobile/android/locales/l10n.toml b/mobile/android/locales/l10n.toml
 --- a/mobile/android/locales/l10n.toml
 +++ b/mobile/android/locales/l10n.toml
 @@ -9,8 +9,9 @@ locales = [
 +    "ab-CD",
      "ar",
      "be",
      "ca",
      "cs",
      "da",
      "es-AR",
@@ -9,8 +9,9 @@ exclude-multi-locale = [
  +    "ab-CD",
       "ia",
       "oc",
...
```

`M` in `hg status` indicates that the file has been modified, `+` in `hg diff` that the line has been added. Follow the instructions available in [this document](../../tools/mercurial/creating_mercurial_patch.md) to create a patch, submit it for review, and land it.

## After the patch has landed

A couple of days after the patch has landed check on FTP if builds are being generated for this new locale: https://ftp.mozilla.org/pub/mobile/nightly/latest-mozilla-central-android-api-16-l10n/

If there are no builds present, it’s usually due to errors in the locale, for example straight single or double quotes in Android DTDs. Check the team’s dashboard for these kind of errors.

[bug template]: https://bugzilla.mozilla.org/enter_bug.cgi?assigned_to=lebedel.delphine%40gmail.com&bug_file_loc=http%3A%2F%2F&bug_ignored=0&bug_severity=normal&bug_status=NEW&cc=francesco.lodolo%40gmail.com&cf_blocking_b2g=---&cf_blocking_fennec=---&cf_fx_iteration=---&cf_fx_points=---&cf_status_b2g_2_0=---&cf_status_b2g_2_0m=---&cf_status_b2g_2_1=---&cf_status_b2g_2_1_s=---&cf_status_b2g_2_2=---&cf_status_b2g_2_2r=---&cf_status_b2g_2_5=---&cf_status_b2g_2_6=---&cf_status_b2g_master=---&cf_status_firefox46=---&cf_status_firefox47=---&cf_status_firefox48=---&cf_status_firefox49=---&cf_status_firefox_esr38=---&cf_status_firefox_esr45=---&cf_tracking_b2g=---&cf_tracking_firefox46=---&cf_tracking_firefox47=---&cf_tracking_firefox48=---&cf_tracking_firefox49=---&cf_tracking_firefox_esr38=---&cf_tracking_firefox_esr45=---&cf_tracking_firefox_relnote=---&cf_tracking_relnote_b2g=---&comment=Please%20add%20the%20LOCALENAME%20%28ab-CD%29%20locale%20to%20all-locales%20to%20start%20Fennec%20single-locale%20builds%20in%20central%20and%20add%20Fennec%20to%20this%20team%27s%20dashboard.&component=General&contenttypemethod=autodetect&contenttypeselection=text%2Fplain&defined_groups=1&flag_type-37=X&flag_type-4=X&flag_type-41=X&flag_type-607=X&flag_type-720=X&flag_type-721=X&flag_type-737=X&flag_type-781=X&flag_type-787=X&flag_type-799=X&flag_type-800=X&flag_type-803=X&flag_type-835=X&flag_type-855=X&flag_type-864=X&flag_type-875=X&flag_type-889=X&flag_type-892=X&flag_type-901=X&flag_type-905=X&flag_type-908=X&form_name=enter_bug&maketemplate=Remember%20values%20as%20bookmarkable%20template&op_sys=Unspecified&priority=--&product=Firefox%20for%20Android&rep_platform=Unspecified&short_desc=%20Add%20%22ab-CD%22%20to%20Fennec%20all-locales%20for%20single-locale%20builds&target_milestone=---&version=unspecified
