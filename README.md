*June 4, 2010: The project configuration has been updated based on user feedback to reduce the number of configuration problems in Eclipse, and to provide better support for those using other build tools, such as Ant.  If you pull this configuration from GitHub, you may need to update your project configuration, or create a new project with your source -- see the set up instructions below.*

This open source Java library allows you to integrate Facebook into your Android application.

Except as otherwise noted, the Facebook Connect Android SDK is licensed under the Apache License, Version 2.0 (http://www.apache.org/licenses/LICENSE-2.0.html)

Alpha Status
============

This is an _alpha_ release. In order to guide the development of the library and allow you to freely inspect and use the source, we have open-sourced the library. The underlying APIs are generally stable, however we may make changes to the library in response to developer feedback.

Known Issues
------------

1. In the Facebook login dialog, the WebKit WebView password field misaligns text input and does not display correctly on Android 2.0 and 2.1.  This is corrected in Android 2.2 (Froyo): see http://code.google.com/p/android/issues/detail?id=5596

2. The example apps do not automatically redraw a dialog if the screen orientation changes.

3. Binary API parameters (such as uploading pictures) is not yet supported -- coming soon, but if you have already implemented it, feel free to send us a patch!

4. The dialog webviews may be blank if an error occurs -- we are working on figuring these out and providing more debugging information.  Sorry for the frustration.

5. If you see "an invalid next or cancel parameter was specified" message in the login dialog, then you need to migrate your application to the New Data Permissions.  This can be done by going to http://www.facebook.com/developers/apps.php then selecting the application you are testing with, and clicking "Edit Settings" (the third item underneath Total Users).  On the settings page, click on Migrations (bottom of the left menu), then set New Data Permissions to "Enabled".

Getting Started
===============

The SDK is lightweight and has no external dependencies. Getting started is quick and easy.

Install necessary packages
--------------------------

* Follow the (http://developer.android.com/sdk/index.html)[Android SDK Getting Started Guide].  You will probably want do set up a device emulator and debugging tools (such as using "adb logcat" for viewing the device debugging and error log).

* Pull the read-only repository from github

     e.g. "git clone git://github.com/facebook/facebook-android-sdk.git"

     (if you have trouble, you could also try "git clone http://github.com/facebook/facebook-android-sdk.git")

To build with Eclipse (3.5), do the following:

* Create a new project for the Facebook SDK in your Eclipse workspace. 
  * Open the __File__ menu, select New --> Project and choose __Android Project__ (inside the Android folder), then click Next.
  * Select "Create project from existing source".
  * Select the __facebook__ subdirectory from within the git repository. 
  * You should see the project properties populated (you might want to change the project name to something like "FacebookSDK").
  * Click Finish to continue.

The Facebook SDK is now configured and ready to go.  

Run the sample application
--------------------------

To test the SDK, you should run the simple sample application.  You can do this with Eclipse (3.5) as follows:

* Create the sample application in your workspace:
  * Repeat as described above, but choose the __examples/simple__ subdirectory from within the git repository.
  * Add your Facebook application ID to the Example.java file.  This Facebook app should use the New Data Permissions, as described in the known issues section above.  If you do not have a Facebook application ID, you can create one: http://www.facebook.com/developers/createapp.php

Build the project: from the Project menu, select "Build Project".  You may see a build error about missing "gen" files, but this should go away when you build the project -- if you have trouble, try running "Clean..." in the Project menu.

Run the application: from the Run menu, select "Run Configurations...".  Under Android Application, you can create a new run configuration: give it a name and select the simple Example project; use the default activity Launch Action.  See http://developer.android.com/guide/developing/eclipse-adt.html#RunConfig for more details.

To run a sample application on a real device, ensure that the device has Internet access, and follow the instructions at http://developer.android.com/guide/developing/device.html 

Create your own application
---------------------------

* Create a Facebook Application: http://www.facebook.com/developers/createapp.php

* Check out the mobile documentation: http://developers.facebook.com/docs/guides/mobile/

* Add a dependency on the Facebook Android SDK library on your application:
  - from the File menu, select "Properties"
  - once the project Properties are displayed, open the Android section, which should list the build targets and libraries
  - in the bottom "Library" section, click "Add..." and select the Facebook SDK project
  - refer to http://developer.android.com/guide/developing/eclipse-adt.html#libraryProject for more details  

* Ensure that your application has network access (android.permission.INTERNET) in the Android manifest:

    <uses-permission android:name="android.permission.INTERNET"></uses-permission>


Usage
=====

With the Android SDK, you can do three main things:

* Authorize users: prompt users to log in to facebook and grant access permission to your application.

User credentials are not handled by the Android application in this SDK: authentication is done in an embedded WebKit WebView using the OAuth 2.0 User-Agent flow to obtain an access token.

* Make API requests

Requests to the Facebook Graph and older APIs are supported in this SDK.  Authenticated requests are done over https using the OAuth access token.

* Display a Facebook dialog

The SDK supports several WebView html dialogs for user interactions, such as creating a wall post.  This is intended to provided quick Facebook functionality without having to implement a native Android UI and pass data to facebook directly though the APIs.

Authentication and Authorization
--------------------------------

User login and application permission requests use the same method: authorize(). By default, if you pass an empty ''permissions'' parameter, then you will get access to the user's basic information., which includes their name, profile picture, list of friends and other general information. For more information, see http://developers.facebook.com/docs/authentication/.

If you pass in extra permissions in the permissions parameter (e.g. "publish_stream", "offline_access"), then the user will be prompted to grant these permissions.  "offline_access" is particularly useful, as it avoids access expiration and ongoing prompts to the user for access.  See http://developers.facebook.com/docs/authentication/permissions 

This SDK uses the (http://tools.ietf.org/html/draft-ietf-oauth-v2)["user-agent"] flow from OAuth 2.0 for authentication details.

To authorize a user, the simplest usage is:

     facebook = new Facebook();
     facebook.authorize(context, applicationId, new String[] {}, new LoginDialogListener());

The authorize method is asynchronous, generating a dialog with WebView content from Facebook, prompting the user to log in and grant access.  The DialogListener is a callback interface that your application must implement: it's methods will be invoked when the dialog process completes or ends in error.

See the sample applications for more specific code samples.

When the user wants to stop using Facebook integration with your application, you can call the logout method to clear all application state and make a server request to invalidate the current OAuth 2.0 token.

     facebook.logout(context);

Accessing the Graph API
-----------------------

The (http://developers.facebook.com/docs/api)[Facebook Graph API] presents a simple, consistent view of the Facebook social graph, uniformly representing objects in the graph (e.g., people, photos, events, and fan pages) and the connections between them (e.g., friend relationships, shared content, and photo tags).

You can access the Graph API by passing the Graph Path to the ''request'' method. For example, to access information about the logged in user, call

    facebook.request("me");               // get information about the currently logged in user
    facebook.request("platform/posts");   // get the posts made by the "platform" page
    facebook.request("me/friends");       // get the logged-in user's friends

The request call is synchronous, meaning it will block the calling thread -- it should not be called from the main (UI) thread in Android. To make it non-blocking, you can make the request in a separate or background thread. For example:

    new Thread() {
      @Override public void run() {
         String resp = request("me");
	 handleResponse(resp);
      }
    }.start();

See the AsyncFacebookRunner class and sample application for examples of making asynchronous requests.

Note that the server response is in JSON string format.  The SDK provides a Util.parseJson() method to convert this to a JSONObject, whose fields and values can be inspected and accessed.  The sample implementation checks for a variety of error conditions and raises JSON or Facebook exceptions if the content is invalid or includes an error generated by the server.  Advanced applications may wish to provide their own parsing and error handling. 

The (http://developers.facebook.com/docs/reference/rest/)[Old REST API] is also supported. To access the older methods, pass in the named parameters and method name as a dictionary Bundle.

    Bundle parameters = new Bundle();
    parameters.putString("method", "auth.expireSession");
    String response = request(parameters);

See the javadoc for the request method for more details.

User Interface Dialogs
----------------------

This SDK provides a method for popping up a Facebook dialog.  The currently supported dialogs are the login and permissions dialogs used in the authorization flow, and a "stream.publish" flow for making a wall post.  The dialog method requires an Android context to run in, an action to perform, and a DialogListener callback interface for notification that must be implemented by the application.  For example,

    facebook.dialog(context, "stream.publish", new SampleDialogListener()); 

This allows you to provide basic Facebook functionality in your application with a singe line of code -- no need to build native dialogs, make API calls, or handle responses.

Error Handling
--------------

For synchronous methods (request), errors are thrown by exception. For the asynchronous methods (dialog, authorize), errors are passed to the onException methods of the listener callback interface.
