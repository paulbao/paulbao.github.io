---
layout: post
title:  "Native Swift as Part of React Native"
date:   2018-10-05 11:42:07 +1100
categories: mobile development
---
We want to integrate an existing swift app as part of React Native app.
This [official document](http://facebook.github.io/react-native/docs/integration-with-existing-apps) is a good instruction to integrate React Native components into existing app. I did some investigation and finally integrated an existing mappedIn sample app into a React Native app.

 - Add MappedIn SDK from CocoaPods
	 Since React Native needs public [CocoaPods](https://github.com/CocoaPods/CocoaPods) specifications, we need to add one more source in Podfile like this:

		source 'https://github.com/MappedIn/podspec.git'
		source 'https://github.com/CocoaPods/Specs.git'

    ...

		pod 'MappedIn', '1.1.0'

 - Make React Native working with Swift
   The MappedIn sample app was written in Swift, and I was trying to expose struct `Mappedin.Service` type to objective-c, but I finally realised that it's not supported. So I decided to run React Native in Swift environment.
   Replaced AppDelegate.h and AppDelegate.m file by a swift file

		@UIApplicationMain
	
		class AppDelegate: UIResponder, UIApplicationDelegate {

		  var window: UIWindow?

		  var bridge: RCTBridge!

		  override init() {

		  super.init()

		  service = Service(AppDelegate.self)
		}

		func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {

		  let jsCodeLocation: URL

		  jsCodeLocation = RCTBundleURLProvider.sharedSettings().jsBundleURL(forBundleRoot: "index", fallbackResource:nil)

		  let rootView = RCTRootView(bundleURL: jsCodeLocation, moduleName: "vicinity", initialProperties: nil, launchOptions: launchOptions)

		  let rootViewController = UIViewController()

		  rootViewController.view = rootView

		  let navigationController = UINavigationController(rootViewController: rootViewController)

		  navigationController.isNavigationBarHidden = true

		  self.window = UIWindow(frame: UIScreen.main.bounds)

		  self.window?.rootViewController = navigationController

		  self.window?.makeKeyAndVisible()

		  return true

		}

    In addition, I created <Project>-Bridging-Header.h to expose RCT modules to swift.

		#import <React/RCTBridgeModule.h>
		#import <React/RCTBridgeModule.h>
		#import <React/RCTBridge.h>
		#import <React/RCTEventDispatcher.h>
		#import <React/RCTRootView.h>
		#import <React/RCTUtils.h>
		#import <React/RCTConvert.h>
		#import <React/RCTBundleURLProvider.h>

 - Create React Native method that expose to JS to start Swift ViewController
 


		  dispatch_async(dispatch_get_main_queue(), ^{

		    AppDelegate *app = (AppDelegate*)[[UIApplication sharedApplication] delegate];

		    UIStoryboard *storyboard = [UIStoryboard storyboardWithName:@"Main" bundle:nil];

		    VenueListController *venueListVC = [storyboard instantiateViewControllerWithIdentifier:@"VenueListViewController"];

		    [((UINavigationController*)app.window.rootViewController) pushViewController:venueListVC animated:YES];

		  });
		}


It takes me a bit of time to initialise `VenueListController` as the tableView that bind with Main storyboard is always nil and this is the only way to make it work.