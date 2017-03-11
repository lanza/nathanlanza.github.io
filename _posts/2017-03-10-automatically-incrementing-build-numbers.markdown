---
layout: post
title:  "Automatically Incrementing Build Numbers"
date:   2017-03-10 12:00:00 -0400
categories: xcode
---

### Build Numbers ###

Your Xcode project's targets each keep track of their current build number. Typical usage would be to increment that number each time you actually build your project. However, Xcode doesn't automatically provide this feature. Also, if you've uploaded an app to the AppStore I'm sure you've forgotten to increment your build number at least once and had to repeat the entire archive and upload process just to change a two to a three in the target's settings.

So I decided to write a shell script that will automatically increment this value for me.

{% highlight bash %}
1   if [[ $INFOPLIST_FILE =~ '^/Users' ]]; then
2     location=$INFOPLIST_FILE
3   else
4     location=${PROJECT_DIR}/${INFOPLIST_FILE}
5   fi
6   getBuildNumber () {
7     echo $(/usr/libexec/PlistBuddy -c "Print CFBundleVersion" "$location")
8     return
9   }
10  buildNumber=$[ $(getBuildNumber) + 1 ]
11  /usr/libexec/PlistBuddy -c "Set :CFBundleVersion $buildNumber" "$location"
{% endhighlight %}

I'll explain this line by line.

#### Lines one through five: ####
`$INFOPLIST_FILE` and `$PROJECT_DIR` are build settings imported into the shell environment for you by Xcode. The former pointing to the location of your project directory and the latter pointing to the location of your info.plist file.

However, if you do some custom configuration to your info.plist file, you could end up changing the build setting. The build setting is normally relative (i.e. `myProject/info.plist`) but I have some projects where the path is absolute (i.e. `/Users/me/projects/myProject/myProject/myProject.plist`). So the first if statement checks if you have an absolute path or a relative one and defines the `location` variable accordingly.

#### Lines six through nine ####
Here we simply define a function `getBuildNumber` that uses Xcode's `PlistBuddy` command line tool. As you can guess from the name, it helps you access plist settings. Check `man PlistBuddy` if you want specifics, but this line gets the `CFBundleVersion` from the plist at `$location` and sends it to stdout. Clearly enough, this just gets the bundle version (aka the build number) from the plist and echos it out to stdout. I declare this function for testing purposes where I call it a handful of times, you can just call this inline if you wish. 

#### Line 10 ####
This line calls the function, adds one to it and stores it to the variable `buildNumber`. Straight forward enough.

#### Line 11 ####
This line just uses the same `PlistBuddy` utility to set the incremented build number.

#### Usage ####
Now, how do you use it? I put this file at `/Users/me/bin/incrementXcodeBuild`. I also include that folder in my path, though it's position their isn't useful as it relies on the injected Xcode environment variables.

Now we have to configure Xcode to use it. Open your .xcproject or .xcworkspace and click the project in the navigator, then click your target and then Build Phases. Then click the "+" and "New Run Script Phase." Expand the "Run Script" entry and type this in the script entry box:

{% highlight bash %}
if [[ $USER == 'yourUsernameHere' ]]; then
  /Users/me/bin/incrementXcodeBuild
fi
{% endhighlight %}

All this that we have just added will run the script at the end of the build process for your app. Build Phases is more or less a list of actions that Xcode does during the build process. So we added a final step of calling this script. 

The script just above first checks if it is being ran on your user account. If so, it calls our `incrementXcodeBuild` script. I perform this check in order to avoid calling this script if the build is being ran on Travis or Xcode Server as I upload via fastlane from my MacBook Pro. 

And that's it! Hope you found this useful!
