---
layout: post
title:  "iOS App Structure"
date:   2016-11-10 12:00:00 -0400
categories: swift, ios
draft: yes
---

Today I decided to write a post about some patterns that I use to structure my apps. I figure some of it will be useful to others while some more advanced iOS developers will surely be able to give me some constructive criticism.

First, a few ideas. I'm not a fan of Storyboard segues whatsoever. Navigation is not a responsibility of the controller and embedding it into a nib to be controlled via dynamic dispatch is even worse. I've started to use Storyboards for any graphical view layout I'd like to do and just instantiating them via `UIStoryboard` methods.

{% highlight swift %}
extension UIStoryboard {
  static var main: UIStoryboard { return UIStoryboard(name: "Main", bundle: nil) }
}
protocol ViewControllerFromStoryboard {
  static var storyboardIdentifier: String { get }
  static func new() -> Self
}
extension ViewControllerFromStoryboard {
  static func new() -> Self { return UIStoryboard.main.instantiateViewController(withIdentifier: storyboardIdentifier) }
}
extension MyViewController: ViewControllerFromStoryboard {
  static var storyboardIdentifier: String { return "MyViewController" }
}
{% endhighlight %}

This is merely syntactic sugar, but it's nice to abstract away the fact that you are working with storyboards from your codebase. Furthermore, I also prefer to take control away from storyboards altogether. Setting up the initial VC should be something done through code from your app's entry point in `AppDelegate`.

{% highlight swift %}
func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {

  let mainViewController = MainViewController.new()
  let mvcNavItem = mainViewController.navigationItem
  //setup navItem
  let mvcTab = mainViewController.tabBarItem
  //setup tabBarItem
  let mvcNav = UINavigationController(rootViewController: mainViewController)

  //make the rest of your tabs

  let tabBarController = UITabBarController()
  tabBarController.viewControllers = [mvcNav, //the rest go here]

  window = UIWindow()
  window!.rootViewController = tabBarController
  window!.makeKeyAndVisible()

  return true
}
{% endhighlight %}

First, I'm not a HUGE fan of creating VCs in `didFinishLaunchingWithOptions` either. The `AppDelegate` is a single purpose class with the purpose of being the `UIApplication`'s delegate. Instantiation of view controllers should be handled elsewhere. I'm particularly fond of the coordinator pattern, but I won't go over that here. I plan on doing a post about that in the future and I'm also working on my own micro-framework for personal use (and perhaps to publish it to cocoapods if it ends up being well enough coded.) But for now, we're kicking off the app via code in `didFinish...`.

Another comment I'd like to make is what `makeKeyAndVisible()` does as I've heard people consider it voodoo and just write it because they are told to. It's really rather simpe. "Make key" means to make it the receipient of keyboard input. The "key" window is the window that receives keyboard/mouse input. The API is the same on macOS development where the "key" window is an important designation but one that is infrequently used on iOS. "and visible" is rather obvious, I assume.

Next, let's say that the first VC we are looking at will have a `UITableView` as it's main view. I use a few different patterns here to best facilitate code reuse, isolation of concerns and some other fancy terms.
