# Using the OAuth 2 Controllers #



## Introduction ##

[OAuth 2](http://wiki.oauth.net/w/page/25236487/OAuth-2) is a protocol allowing your application to obtain authorization to read or modify a user’s files or data on an external server.

The server generates a web page for the user to sign in with her name and password, including a button explicitly granting access to some of her data. Upon successful authentication, the server gives tokens to your application representing the user's authorization.

With the Objective-C OAuth 2 controllers, the web page can be presented as an iOS view or a Mac sheet within your application. The controllers also provide an authentication object that simplifies your application's future requests for the user's data.

There is also a [gtm-oauth](http://code.google.com/p/gtm-oauth) library for the older OAuth 1 protocol. Most applications should use the OAuth 2 library for services that support OAuth 2.

## Using the iOS and Mac OS X OAuth 2 Controllers ##

The OAuth 2 controllers are useful for authenticating both to Google servers and to other services.

There are example iOS and Mac applications using the OAuth 2 controllers in the library's [Examples directory](http://code.google.com/p/gtm-oauth2/source/browse/#svn/trunk/Examples).

Controller methods with "Google" in the name are for use only with Google services.

## Adding the Controllers to Your Project ##

The project has targets for building a static library for iOS and a framework for Mac OS X. Alternatively, the source files and xibs may be dragged directly into your project file and compiled with your application.

Check out the "top-of-trunk" OAuth 2 controller sources with a Mac terminal window and the command on the [source checkout page](http://code.google.com/p/gtm-oauth2/source/checkout).
The source files required are:

|_iOS and Mac OS X_|_iOS_|_Mac OS X_|
|:-----------------|:----|:---------|
|GTMOAuth2Authentication.h/m<br>GTMOAuth2SignIn.h/m<br>GTMHTTPFetcher.h/m<table><thead><th>GTMOAuth2ViewControllerTouch.h/m<br>GTMOAuth2ViewTouch.xib</th><th>GTMHTTPFetchHistory.h/m<br>GTMOAuth2WindowController.h/m<br>GTMOAuth2Window.xib</th></thead><tbody></tbody></table>

These source files can be browsed in the project's <a href='http://code.google.com/p/gtm-oauth2/source/browse/#svn/trunk/Source/'>source directory</a>.<br>
<br>
Add the <a href='https://developer.apple.com/library/ios/#recipes/xcode_help-project_editor/Articles/AddingaLibrarytoaTarget.html'>standard frameworks</a> Security.framework and SystemConfiguration.framework to your project.<br>
<br>
When linking against a static library build of the controller, specify the <a href='http://developer.apple.com/mac/library/qa/qa2006/qa1490.html'>-ObjC build option</a> for the application target's "Other Linker Flags".<br>
<br>
<h3>ARC Compatibility</h3>

When the controller source files are compiled directly into a project that has ARC enabled, then ARC must be disabled specifically for the controller source files.<br>
<br>
To disable ARC for source files in Xcode 4, select the project and the target in Xcode. Under the target "Build Phases" tab, expand the Compile Sources build phase, select the library source files, then press Enter to open an edit field, and type <code>-fno-objc-arc</code>  as the compiler flag for those files.<br>
<br>
<h3>System Requirements</h3>

The Mac controller is compatible with Mac OS X 10.5 and later. The iOS controller is compatible with iPhone OS 3 and later.<br>
<br>
The sample applications use Objective-C blocks, so are intended for iOS 4 and Mac OS X 10.6.<br>
<br>
<h3>Threading and Queues</h3>

Sign-in calls should be made on the main thread. Authorizing a request is best done on the main thread as well.<br>
<br>
Authorizing a request may be done on a background thread if the thread has a spinning run loop. Alternatively, a GTMOAuth2Authentication object may be assigned a GTMHTTPFetcherService object, where the fetcher service object has a delegateQueue property indicating the queue to be used for calling back to the application. Note that delegateQueue support requires iOS 6 or Mac OS X 10.7.<br>
<br>
<h2>Signing In to Google Services</h2>

<h3>Registering Your Application</h3>

To sign in to Google services with the Objective-C OAuth 2 controllers, first obtain a client ID and secret from the <a href='https://console.developers.google.com/'>API Console</a>

In the console, create a project, then click "API Access" in the left column.  Create a new Client ID for an <b>Installed Application</b> (not a web application) type <b>other</b> (not iOS).  This will provide Client ID and Secret strings to be used with the controller.<br>
<br>
Click "Services" in the left column, and check the list of APIs to verify that the needed APIs are turned on for the project.  Any Google APIs not listed in the console do not need to be turned on individually.<br>
<br>
<h3>Displaying the Controllers</h3>

<b>iOS:</b> To display a sign-in view, your iOS application makes these calls to push the view:<br>
<pre><code>#import "GTMOAuth2ViewControllerTouch.h"<br>
<br>
static NSString *const kKeychainItemName = @"OAuth2 Sample: Google+";<br>
<br>
NSString *kMyClientID = @"abcd";     // pre-assigned by service<br>
NSString *kMyClientSecret = @"efgh"; // pre-assigned by service<br>
<br>
NSString *scope = @"https://www.googleapis.com/auth/plus.me"; // scope for Google+ API<br>
<br>
GTMOAuth2ViewControllerTouch *viewController;<br>
viewController = [[[GTMOAuth2ViewControllerTouch alloc] initWithScope:scope<br>
             clientID:kMyClientID<br>
         clientSecret:kMyClientSecret<br>
     keychainItemName:kKeychainItemName<br>
             delegate:self<br>
     finishedSelector:@selector(viewController:finishedWithAuth:error:)] autorelease];<br>
<br>
[[self navigationController] pushViewController:controller<br>
                                       animated:YES];<br>
</code></pre>

Each service will provide developers with <b>Client ID</b> and <b>Client Secret</b> strings intended to be hardcoded as constants into applications.  Google services issue the strings via the <a href='https://console.developers.google.com/'>Google API Console</a>.<br>
<br>
<b>Mac:</b> A Mac application would display sign-in as a sheet on the current window, like this:<br>
<br>
<pre><code>#import "GTMOAuthWindowController.h"<br>
<br>
GTMOAuth2WindowController *windowController;<br>
windowController = [[[GTMOAuth2WindowController alloc] initWithScope:scope<br>
              clientID:kMyClientID<br>
          clientSecret:kMylientSecret<br>
      keychainItemName:kKeychainItemName<br>
        resourceBundle:nil] autorelease];<br>
<br>
[controller signInSheetModalForWindow:currentWindow<br>
                             delegate:self<br>
                     finishedSelector:@selector(windowController:finishedWithAuth:error:)];<br>
</code></pre>
The <b>scope</b> is a string identifying what access is being requested. For access to more than one scope, separate the scopes with a space, or use the convenience method <code>scopeWithStrings:</code> in the class  <code>GTMOAuth2Authentication</code>.<br>
<br>
The <b>keychain item name</b> is used to save the token on the user’s keychain, and should identify both your application name and the service name(s). If keychainItemName is nil, the token will not be saved, and the user will have to sign in again the next time the application is run.<br>
<br>
When the user signs in successfully or cancels signing in, the view or window controller will invoke your finishedSelector’s method:<br>
<pre><code>- (void)viewController:(GTMOAuth2ViewControllerTouch *)viewController<br>
      finishedWithAuth:(GTMOAuth2Authentication *)auth<br>
                 error:(NSError *)error {<br>
 if (error != nil) {<br>
   // Authentication failed<br>
 } else {<br>
   // Authentication succeeded<br>
 }<br>
}<br>
</code></pre>

If <code>[error code]</code> is kGTMOAuthErrorWindowClosed (-1000), then the user closed the sign-in view before completing authentication. Otherwise, any error reflects the server response in validating the user's access.<br>
<br>
The controllers also support Objective-C block completion handlers as alternatives to the delegate and finished selectors.<br>
<br>
<h3>Using the Authentication Tokens</h3>

If authentication succeeds, your application should retain the authentication object. It can be used directly to authorize NSMutableURLRequest objects:<br>
<br>
<pre><code>[auth authorizeRequest:myNSURLMutableRequest<br>
              delegate:self<br>
     didFinishSelector:@selector(authentication:request:finishedWithError:)];<br>
</code></pre>

The authentication object may need to fetch a refreshed access token before authorizing the request, so the authorization call is asynchronous, though it will often authorize the request and call the callback immediately.<br>
<br>
If the error passed to the callback is nil, then the request was authorized:<br>
<pre><code>- (void)authentication:(GTMOAuth2Authentication *)auth<br>
               request:(NSMutableURLRequest *)request<br>
     finishedWithError:(NSError *)error {<br>
 if (error != nil) {<br>
   // Authorization failed<br>
 } else {<br>
   // Authorization succeeded<br>
 }<br>
}<br>
</code></pre>

Authorization may also be done with a block callback:<br>
<pre><code>[auth authorizeRequest:myNSURLMutableRequest<br>
     completionHandler:^(NSError *error) {<br>
  if (error == nil) {<br>
    // the request has been authorized<br>
  }<br>
}];<br>
</code></pre>

With the <a href='http://code.google.com/p/gtm-http-fetcher/'>GTM HTTP Fetcher</a>, the authentication object can be attached to the fetcher, making the authorization step transparent:<br>
<br>
<pre><code>[fetcher setAuthorizer:auth];<br>
</code></pre>

Similarly, in the <a href='http://code.google.com/p/google-api-objectivec-client/'>Google APIs Client library</a> and the <a href='http://code.google.com/p/gdata-objectivec-client/'>Google Data APIs library</a> the authentication object can be stored in a library service object to authorize future API requests:<br>
<br>
<pre><code>[[self contactService] setAuthorizer:auth];<br>
</code></pre>

<b>Note:</b> To avoid sending authorization tokens insecurely, authorization should be added only to requests with the https scheme. To authorize a request that has any other URL scheme, the authentication object property <code>shouldAuthorizeAllRequests</code> must first be set to YES to override the https requirement and avoid the error <code>kGTMOAuth2ErrorUnauthorizableRequest</code>.<br>
<br>
<h3>Retrieving Authorization from the Keychain</h3>

If your application saves the authorization to the keychain (by setting the controller's keychainItemName), it can be retrieved the next time the application launches:<br>
<br>
<pre><code>- (void)awakeFromNib {<br>
  // Get the saved authentication, if any, from the keychain.<br>
  GTMOAuth2Authentication *auth;<br>
  auth = [GTMOAuth2ViewControllerTouch authForGoogleFromKeychainForName:kKeychainItemName<br>
                                                               clientID:kMyClientID<br>
                                                           clientSecret:kMyClientSecret];<br>
<br>
  // Retain the authentication object, which holds the auth tokens<br>
  //<br>
  // We can determine later if the auth object contains an access token<br>
  // by calling its -canAuthorize method<br>
  [self setAuthentication:auth];<br>
}<br>
</code></pre>

If no authorization was saved, then “auth” will still be a valid authorization object but will be unable to authorize requests:<br>
<br>
<pre><code>BOOL isSignedIn = [auth canAuthorize]; // returns NO if auth cannot authorize requests<br>
</code></pre>

<h3>Signing Out</h3>

To completely discard the user’s authorization, use the view or window controller calls to remove the keychain entry and to ask the Google server to revoke the token:<br>
<pre><code> [GTMOAuth2ViewControllerTouch removeAuthFromKeychainForName:kKeychainItemName];<br>
<br>
 [GTMOAuth2ViewControllerTouch revokeTokenForGoogleAuthentication:auth];<br>
</code></pre>

Finally, release the authorization object.<br>
<br>
<h2>Signing in to non-Google Services</h2>

<h3>Registering Your Application</h3>

Check the API documentation for the service to determine how to obtain a client ID and secret.<br>
<br>
If the service supports installed applications with the redirect URI "urn:ietf:wg:oauth:2.0:oob", then register your application as an installed application.<br>
<br>
Otherwise, register as a normal web application, and if the registration requires that you also specify a redirect URI, supply some <b>invalid</b> URL at your domain. The controllers will look for loading of the redirect URI as an indication that sign-in has completed, but will not attempt to load the URL.<br>
<br>
<h3>Displaying the Controllers</h3>


To sign in to a non-Google service, you should consult the service's API documentation to obtain client ID and client secret strings, to find the required scope string for API operations, and to determine the API endpoints (URLs) for authorization and for obtaining tokens.<br>
<br>
To use the OAuth 2 controllers with services other than Google APIs, your application should create an authentication object with the appropriate constants, like this:<br>
<pre><code>NSString *kMyClientID = @"abcd";     // pre-assigned by service<br>
NSString *kMyClientSecret = @"efgh"; // pre-assigned by service<br>
<br>
- (GTMOAuth2Authentication *)myCustomAuth {<br>
<br>
  NSURL *tokenURL = [NSURL URLWithString:@"https://api.example.com/oauth/token"];<br>
<br>
  // We'll make up an arbitrary redirectURI.  The controller will watch for<br>
  // the server to redirect the web view to this URI, but this URI will not be<br>
  // loaded, so it need not be for any actual web page.<br>
  NSString *redirectURI = @"http://www.google.com/OAuthCallback";<br>
<br>
  GTMOAuth2Authentication *auth;<br>
  auth = [GTMOAuth2Authentication authenticationWithServiceProvider:@"Custom Service"<br>
                                                           tokenURL:tokenURL<br>
                                                        redirectURI:redirectURI<br>
                                                           clientID:kClientID<br>
                                                       clientSecret:kClientSecret];<br>
  return auth;<br>
}<br>
</code></pre>

Displaying the sign-in view with a custom auth also requires providing the OAuth 2 endpoints (URLs) and scope string to the controller, as shown here:<br>
<br>
<pre><code>- (void)signInToCustomService {<br>
  [self signOut];<br>
<br>
  GTMOAuth2Authentication *auth = [self authForCustomService];<br>
<br>
  // Specify the appropriate scope string, if any, according to the service's API documentation<br>
  auth.scope = @"read";<br>
<br>
  NSURL *authURL = [NSURL URLWithString:@"https://api.example.com/oauth/authorize"];<br>
<br>
  // Display the authentication view<br>
  GTMOAuth2ViewControllerTouch *viewController;<br>
  viewController = [[[GTMOAuth2ViewControllerTouch alloc] initWithAuthentication:auth<br>
        authorizationURL:authURL<br>
        keychainItemName:kKeychainItemName<br>
                delegate:self<br>
        finishedSelector:@selector(viewController:finishedWithAuth:error:)] autorelease];<br>
<br>
  // Now push our sign-in view<br>
  [[self navigationController] pushViewController:viewController animated:YES];<br>
}<br>
</code></pre>

The keychain item name is used to save the token on the user’s keychain, and should identify both your application name and the service name(s). If keychainItemName is nil, the token will not be saved, and the user will have to sign in again the next time the application is run.<br>
<br>
When the user signs in successfully or cancels signing in, the view or window controller will invoke your finishedSelector’s method:<br>
<pre><code>- (void)viewController:(GTMOAuthViewControllerTouch *)viewController<br>
      finishedWithAuth:(GTMOAuthAuthentication *)auth<br>
                 error:(NSError *)error {<br>
 if (error != nil) {<br>
   // Sign-in failed<br>
 } else {<br>
   // Sign-in succeeded<br>
 }<br>
}<br>
</code></pre>

If <code>[error code]</code> is kGTMOAuth2ErrorWindowClosed (-1000), then the user closed the sign-in view before completing authorization. Otherwise, any error reflects the server response in validating the user's access.<br>
<br>
The controllers also support Objective-C block completion handlers as alternatives to the delegate and finished selectors.<br>
<br>
<h3>Using the Authentication Tokens</h3>

If authentication succeeds, your application should retain the authentication object. It can be used directly to authorize NSMutableURLRequest objects:<br>
<br>
<pre><code>[auth authorizeRequest:myNSURLMutableRequest<br>
              delegate:self<br>
     didFinishSelector:@selector(authentication:request:finishedWithError:)];<br>
</code></pre>

The authentication object may need to fetch a refreshed access token, so the authorization call is asynchronous, though it will often authorize the request and call the callback immediately.<br>
<br>
If the error passed to the callback is nil, then the request was authorized:<br>
<pre><code>- (void)authentication:(GTMOAuth2Authentication *)auth<br>
               request:(NSMutableURLRequest *)request<br>
     finishedWithError:(NSError *)error {<br>
 if (error != nil) {<br>
   // Authorization failed<br>
 } else {<br>
   // Authorization succeeded<br>
 }<br>
}<br>
</code></pre>

Authorization may also be done with a block callback:<br>
<pre><code>[auth authorizeRequest:myNSURLMutableRequest<br>
     completionHandler:^(NSError *error) {<br>
  if (error == nil) {<br>
    // the request has been authorized<br>
  }<br>
}];<br>
</code></pre>

With the <a href='http://code.google.com/p/gtm-http-fetcher/'>GTM HTTP Fetcher</a>, the authentication object can be attached to the fetcher, making the authorization step transparent:<br>
<br>
<pre><code>[fetcher setAuthorizer:auth];<br>
</code></pre>

<b>Note:</b> To avoid sending authorization tokens insecurely, authorization should be added only to requests with the https scheme. To authorize a request that has any other URL scheme, the authentication object property <code>shouldAuthorizeAllRequests</code> must first be set to YES to override the https requirement and avoid the error <code>kGTMOAuth2ErrorUnauthorizableRequest</code>.<br>
<br>
<h3>Retrieving Authorization from the Keychain</h3>

If your application saves the authorization to the keychain (by setting the controller's keychainItemName), it can be retrieved the next time the application launches:<br>
<br>
<pre><code>- (void)awakeFromNib {<br>
  // Get the saved authentication, if any, from the keychain.<br>
  GTMOAuth2Authentication *auth = [self myCustomAuth];<br>
  if (auth) {<br>
    BOOL didAuth = [GTMOAuth2ViewControllerTouch authorizeFromKeychainForName:@"My App: Custom Service"<br>
                                                               authentication:auth];<br>
  }<br>
<br>
  // Retain the authentication object, which holds the auth tokens<br>
  //<br>
  // We can determine later if the auth object contains an access token<br>
  // by calling its -canAuthorize method<br>
  [self setAuthentication:auth];<br>
}<br>
</code></pre>

If no authorization was saved, then “auth” will still be a valid authorization object but will be unable to authorize requests:<br>
<br>
<pre><code>BOOL isSignedIn = [auth canAuthorize]; // returns NO if auth cannot authorize requests<br>
</code></pre>

<h3>Signing Out</h3>

To completely discard the user’s authorization, use the view or window controller calls to remove the keychain entry:<br>
<pre><code>[GTMOAuthViewControllerTouch removeAuthFromKeychainForName:kKeychainItemName];<br>
</code></pre>

Finally, release the authorization object.<br>
<br>
<br>
<h1>Design Notes</h1>

The library's classes are designed in three layers.<br>
<br>
<table><thead><th>Window/View Controller</th><th>user interface & application API, including keychain</th></thead><tbody>
<tr><td>Sign-In               </td><td>OAuth sign-in sequence & WebView handling           </td></tr>
<tr><td>Authentication        </td><td>data handling, token fetches                        </td></tr></tbody></table>

Classes are written to be independent of the layers above them.<br>
<br>
The window and view controllers are retained only during the user's sign-in interaction.<br>
<br>
The sign-in object is typically not used directly by the client application.<br>
<br>
The authentication object must be retained by the client app to authorize NSMutableURLRequests. It is also used to save authentication to and read authentication from the keychain.<br>
<br>
<h1>Questions and Comments</h1>

You can learn more about the OAuth 2 protocol for desktop and mobile applications at <a href='http://code.google.com/apis/accounts/docs/OAuth2.html'>Google's documentation</a>.<br>
<br>
Additional documentation for the controllers is available in the header files.<br>
<br>
If you have any questions or comments about the library or this documentation, please join the <a href='http://groups.google.com/group/gtm-oauth2'>discussion group</a>.