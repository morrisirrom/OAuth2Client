# OAuth2Client

A OAuth2 framework for Mac OS X & iOS (Cocoa & Cocoa touch).

*README will be updated in the next days.*

## Quickstart

### Get the sources

It's as easy as 

- git clone git://github.com/nxtbgthng/OAuth2Client.git

### Include the library in your Xcode project

#### iOS projects

- drag the OAuth2Client.xcodeproj into your project
- add OAuth2Client as a build dependency
- add "/tmp/OAuth2Client.dst/usr/local/include" to your user header search path in the build settings
- link your target against OAuth2Client (drag the OAuth2Client product from OAuth2Client.xcodeproj to your targets *Link Binary With Libraries*)
- #import "NXOAuth2.h"

#### Desktop Mac projects

- drag the OAuth2Client.xcodeproj into your project
- add OAuth2Client.framework as a build dependency
- add "$(CONFIGURATION_BUILD_DIR)/$(CONTENTS_FOLDER_PATH)/Frameworks" to your targets Framework Search Path
- link your target against OAuth2Client (drag the OAuth2Client.framework product from OAuth2Client.xcodeproj to your targets *Link Binary With Libraries*)
- #import &lt;OAuth2Client/NXOAuth2.h&gt;


### Use the OAuth2Client

#### Create an instance of OAuth2Client

To create the OAuth2Client you need OAuth2 credentials (client id & secret) and endpoints (authorize & token URL) for your application. You usually get them from the service you want to connect to. You also need to pass in an *authDelegate* which is discussed later.

<pre>
	// client is a ivar
	client = [[NXOAuth2Client alloc] initWithClientID:@"xXxXxXxXxXxXxXxXxXxXxXxXxXxXxXx"
									 	 clientSecret:@"xXxXxXxXxXxXxXxXxXxXxXxXxXxXxXx"
									 	 authorizeURL:[NSURL URLWithString:@"https://myHost/oauth2/authenticate"]
										 	 tokenURL:[NSURL URLWithString:@"https://myHost/oauth2/token"]
									 	 authDelegate:self];
</pre>

- Start authentication flow

<pre>
	[client requestAuthentication];
</pre>

#### The Authentication Delegate

The Authentication Delegate is the place to get callbacks on the status of authentication. It defines following methods:

<pre>
	- (void)oauthClientDidGetAccessToken:(NXOAuth2Client *)client;
	- (void)oauthClientDidLoseAccessToken:(NXOAuth2Client *)client;
	- (void)oauthClient:(NXOAuth2Client *)client didFailToGetAccessTokenWithError:(NSError *)error;

	- (void)oauthClientRequestedAuthorization:(NXOAuth2Client *)client;
</pre>

The first three inform you when authentication is gained or lost, as well as when an error occurred during the process. The fourth method needs to be implemented by your app and is responsible for choosing the OAuth2 flow. The wrapper supports the web server & the user credentials flow of OAuth2 draft 10. Implement is as following:

<pre>
	- (void)oauthClientRequestedAuthorization:(NXOAuth2Client *)aClient;
	{
		// webserver flow
		NSURL *authorizationURL = [client authorizationURLWithRedirectURL:[NSURL URLWithString:@"x-myapp://oauth2"]];	// this is your redirect url. register it with your app
		#if TARGET_OS_IPHONE
		[[UIApplication sharedApplication] openURL:authorizationURL];	// this line quits the application or puts it to the background
		#else
		[[NSWorkspace sharedWorkspace] openURL:authorizationURL];
		#endif
		
		// OR
		
		// user credentials flow
		[client authorizeWithUsername:username password:password];
		// you probably don't yet have username & password.
		// if so, open a view to query them from the user & call this method with the results asynchronously.
	}
</pre>

#### Sending requests

Create your request as usual but don't use NSURLConnection but NXOAuth2Connection. It has a similar delegate protocol but signs the request when an OAuth2Client is passed in. If you don't pass in the client but nil, the connection will work standalone but not sign any request. Make sure to retain the connection for as long as it's running. The best place for doing so is it's delegate. You can also cancel the connection if you deallocate the delegate.

<pre>
	NSURLRequest *request = [NSURLRequest requestWithURL:[NSURL URLWithString:@"https://serviceHost/resource"]];
	// retain the connection for as long as it's running.
	NXOAuth2Connection *connection = [[NXOAuth2Connection alloc] initWithRequest:request oauthClient:aClient delegate:self];
</pre>

## Known Issues

- Error handling needs to be improved

## BSD License 

Copyright © 2010, nxtbgthng UG (haftungsbeschränkt)

All rights reserved.

Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions are met:

* Redistributions of source code must retain the above copyright
  notice, this list of conditions and the following disclaimer.
* Redistributions in binary form must reproduce the above copyright
  notice, this list of conditions and the following disclaimer in the
  documentation and/or other materials provided with the distribution.
* Neither the name of nxtbgthng nor the
  names of its contributors may be used to endorse or promote products
  derived from this software without specific prior written permission.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND
ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED
WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
DISCLAIMED. IN NO EVENT SHALL <COPYRIGHT HOLDER> BE LIABLE FOR ANY
DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES
(INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES;
LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND
ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
(INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS
SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.