# FileChooser

Simple mission file chooser for Unity AR app to launch.
This app does _**Not**_ use Unity; instead using Android branch.


## Features
* Hardcoded paths:
  * Selected/main mission file will be: /sdcard/Missions.csv
* Expected file format:
  * not checked.
  * CSV, comma-separated. Team members are semi-colon (;) separated
* File chooser for mission CSV files.
  * It copies chosen file and overrides the main file.
  * Sends result (file path) back to main app now, and everyone in future.

## Issues

### Solved / Notes
* Unity app can launch Android app/activity. Sample code:
```
        public void LaunchExtActivity()
        {
            bool fail = false;
            string bundleId = "org.ds.filechooser";
            AndroidJavaClass up = new AndroidJavaClass("com.unity3d.player.UnityPlayer");

            AndroidJavaObject ca = up.GetStatic<AndroidJavaObject>("currentActivity");
            AndroidJavaObject packageManager = ca.Call<AndroidJavaObject>("getPackageManager");

            AndroidJavaObject launchIntent = null;
            try
            {
                launchIntent = packageManager.Call<AndroidJavaObject>("getLaunchIntentForPackage", bundleId);
            }
            catch (System.Exception e)
            {
                fail = true;
            }

            if (fail)
            { //open app in store
                Application.OpenURL("https://google.com");
            }
            else //I want to open Gallery App? But what activity?
            {
                try
                {
                    ca.Call("startActivity", launchIntent);
                    //ca.Call("startActivityForResult", launchIntent); //no such method "startActivityForResult"
                    WriteToCustomConsole("Called intent");
                }
                catch (System.Exception e)
                {
                    WriteToCustomConsole(string.Format("XXX Failed to start external activity!") + e.Message);
                }
            }
            WriteToCustomConsole(string.Format("Called external activity."));

            up.Dispose();
            ca.Dispose();
            packageManager.Dispose();
            launchIntent.Dispose();
        }

	//Example, call LaunchExtActivity() in Assets\JK\Scripts\CustomAugImgController.cs
	public void Update()
	{
		...
		WriteToCustomConsole(string.Format("Main anchor instantiated!"));
                LaunchExtActivity();
                ...
```
  * The `bundleId` should be the Android app's package name, not the class name.
  * The 1st get 'AndroidJavaClass' is always "com.unity3d.player.UnityPlayer", unless you wrote your own derivative class.
    See [Unsolved] section.
    * Unity app seems to only be able to launch the **main** activity in an app. Haven't figured out how to specify a non-main activity - something easily do-able in Android apps.
      * Also haven't figured out how to activate a custom _intent_. That's why sample code is just activating default launch main activity intent, i.e. `getLaunchIntentForPackage`.
  * The `Application.OpenURL(<url>)` statement actually works - it's an alternate way to launch default Android apps.
* In newer Android API versions, need to include code to explcitly request for permissions (e.g.: READ_EXTERNAL_STORAGE), besides declaring in manifest file.
  * Refer to sample code in `onCreate`.
  * If not, you could go to app settings to _**manually** allow storage permissions_.

### Unsolved
* Unity app C+ cannot easily retrieve Android app/activity return value.
  * In Android app, it's a simple matter of calling `startActivityForResult`
  and adding a listener `onActivityResult` to receive the results.
  * For Unity, supposedly need to write a custom (inherited) Java activity to receive the results - then might as well use Java completely `=\`
  See https://docs.unity3d.com/Manual/AndroidUnityPlayerActivity.html
  * For now, Unity app can manually check for changes to /sdcard/Missions.csv when app resumes?