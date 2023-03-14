# Determining Mobile Element Selectors in Appium

*Just as with websites, tools exist to inspect native apps. We can use these tools, of which Appium Inspector is our favorite example, to help find out which selectors we can use to find specific elements for our automation code.*

## Element discovery: Getting app source

Let's learn how to decide on locator strategies and selectors for finding elements in our mobile apps. There are two basic ways to do this. I'm going to show you one way first, even though the second way is much better and easier. So what's the first way, and why should we care about it? The first way to discover locators is to simply print out the page source and examine it yourself, learning from the various element properties what you need to know in order to find a given element.

This is an important skill to learn because it's one you may need to fall back on at some point. In certain circumstances or environments you might not have access to the tools that make your life easier in determining selectors. But if you can run an Appium session, you can get the page source! In fact, *examining the page source is a useful debugging technique, not just a technique for finding elements*.

To illustrate this I'm going to use file [<code>source_ios.py</code>](https://github.com/lana-20/appium-element-discovery/blob/main/source_ios.py) as a code sample:

    import time
    from os import path
    from appium import webdriver

    CUR_DIR = path.dirname(path.abspath(__file__))
    APP = path.join(CUR_DIR, 'TheApp.app.zip')
    APPIUM = 'http://localhost:4723'
    CAPS = {
        'platformName': 'iOS',
        'platformVersion': '16.2',
        'deviceName': 'iPhone 14 Pro',
        'automationName': 'XCUITest',
        'app': APP,
    }

    driver = webdriver.Remote(APPIUM, CAPS)
    try:
        time.sleep(4)
        print(driver.page_source)
    finally:
        driver.quit()

What we want to do is print out the page source:

    print(driver.page_source)

This is great, but it might not work reliably yet. Why not? Well, this command will be executed as soon as the Appium session starts. But what if the app is still getting itself ready at this time? You might recall seeing a short splash screen show up for the app. What if our page source command gets executed at this point? It will still produce some kind of output, but it might not be output that represents the actual home view of the app. Normally to get around this kind of potential race condition we would use a WebDriverWait. But right now, we don't have any elements we know we can wait for! We're pretending right now that we don't know any element selectors yet, so we can't wait for an element either. For this reason, I'm going to temporarily use a static wait, just to make sure the app has had time to load. Static waits aren't great, but after all, I'm not using it as part of an actual test. I'm just using it to help develop my test, and that's just fine.

The way I'll implement this static wait is first by importing the Python <code>time</code> module up top:

        import time

Now, immediately before the page source command line command, I'll use the time.sleep function:

        time.sleep(4)

I think a value of 4 seconds is plenty of time to be sure the splash screen has gone away. OK, let's run this. I'll switch to my terminal, where I'm in the mobile directory, and run the script using Python:

        python source_ios.py

Or, on macOS

        python3 source_ios.py

Once this is complete, what do we see? The script has finished, having printed out a bunch of stuff here. 

<img width="1000" src="https://user-images.githubusercontent.com/70295997/224857879-0781d8fb-6418-41c5-b885-f002d45a0bac.png">

This is an XML document representing the structure of the UI at the time we requested the page source. Every element that Appium knows about is represented as an XML node here. So if you don't see an element in this document, then Appium will not be able to find it as a normal UI element. Let's look down through this for a bit. I see that each element has a type, like <code>XCUIElementTypeStaticText</code> or <code>XCUIElementTypeOther</code>. There's not too many different types, really, so they're not necessarily all that helpful. But each element has a set of attributes, which can be pretty helpful. For example, I see this section here which appears to show the list of views in my app. Looking at the <code>name</code> attribute, I see "Echo Box", "Login Screen", and so on. And this <code>name</code> attribute is actually quite important, since on iOS, if an element has a <code>name</code> attribute, that means we can find that element using the Accessibility ID strategy, and for our selector using the value of this <code>name</code> attribute. So looking at this source, I could find any of these elements like Echo Box or Login Screen simply by using the Accessibility ID strategy.

<img width="1000" src="https://user-images.githubusercontent.com/70295997/224858535-214b8451-3450-44be-9105-38d30de914eb.png">


As you can imagine, looking at the XML output here is also useful for coming up with XPath selectors, because when you run an XPath query to find an element, it is precisely this document which is consulted in order to do so! So once you've learned a bit more advanced XPath, you can walk through this document and figure out what a good XPath query might be for a particular element in the document that you want to find with Appium. This is useful, but not particularly fun or easy.   

## Element discovery: [Appium inspector](https://youtu.be/PG0PiHnukXA)

Let's now try the selector discovery strategy that I typically use when I have the option - *Appium Inspector*. The Inspector used to be built into the Appium Desktop. We had to launch the Server GUI app, go up to the "File" menu (on Windows) or to the "Appium" menu (on Mac) and then click "New Session". It opened up a window that gave you a graphical interface for launching Appium sessions. Now the Inspector is its own app:

<img width="400" src="https://user-images.githubusercontent.com/70295997/224852350-45139cc7-6a5f-44c8-b18e-d5ae8c2e27c9.png">

Appium Inspector is released in two formats:
1. As a desktop app for macOS, Windows, and Linux. You can get the most recent published version of this app at the [Releases](https://github.com/appium/appium-inspector/releases) section of this repo. Simply grab the appropriate version for your OS and follow standard installation procedures (but see the note below for macOS).
2. As a [web application](https://inspector.appiumpro.com/), hosted by [Appium Pro](https://appiumpro.com/). (It's currently a [known issue](https://github.com/appium/appium-inspector/issues/103) that the web version does not work on Safari). Please make sure to read the note on CORS as well.

Both apps have the exact same set of features, so you might find that simply opening the web version is going to be easier and save you on disk space (and you can keep multiple tabs open!).

Download and open Appium Inspector, populate the <code>Remote Host</code> and <code>Remote Port</code> fields, as well as add your capabilities one by one under the <code>Desired Capabilities</code> tab.

<img width="800" src="https://user-images.githubusercontent.com/70295997/224860839-9b80ed3a-8f7d-4c39-9ede-ae060b047e3c.png">

The same way that we define an Appium server location and a set of capabilities in code, we can do it here, only by clicking and typing. So let's figure out how to configure this so that we can start a session on our test app. First, I'm going to look at the Server configuration portion. I want to make sure that "Appium Server" tab is selected, because I'm going to talk to the Appium server I already have running on the command line. So for Remote Host I'm going to make sure it's <code>localhost</code>, and for port, <code>4723</code>. And for Remote Path, it should just be <code>/</code> because my Appium server is hosting everything directly on the server, with no extra path prefix.

Now I'm looking at the Desired Capabilities tab on the bottom. I can use this interface to add, edit, and remove capabilities. Let's start setting up the capabilities for my app. I just start filling them out, but if we need any reminders, we can always look at what we have in our Python script for iOS for axample, since we just want the same capabilities as those.

- <code>platformName</code> is <code>iOS</code>
- <code>platformVersion</code> is <code>16.2</code> (make sure you use whichever one youv'e been using).
- <code>deviceName</code> is <code>iPhone 14 Pro</code>
- <code>automationName</code> is <code>XCUITest</code>
- And for <code>app</code>, I can actually change the type from text to filepath, and I'm able to use a native file picker dialog to find the path to my <code>TheApp.app.zip</code> file.

Once I've set these, my capabilities are complete. But I don't want to have to do this again, so I can save the capabilities as a set by clicking 'Save As...', and giving them a name. Now, if I want to use these same capabilities in the future, I can just go over to the Saved Capability Sets tab, find the one I want, and start a session.

So now I'm ready to start a session, and I'll click the button to get going. At this point, a new session is starting on our Appium server. If we look at the Appium server logs, we can see that they are doing things and handling our session just as though we were running a new sesssion request from code.

<img width="800" src="https://user-images.githubusercontent.com/70295997/224861934-af90b318-9438-49a4-9bb0-df42815a3688.png">


It's not being started by Python anymore, but rather is being started by Appium Inspector, which is in itself a graphical Appium client. Once the session is up and loaded, I'm greeted with the Appium Inspector UI. In this UI, there are three main sections. On the left we have a screenshot of the app. In the middle we have the app hierarchy as an XML document that we can actually navigate through. And on the right, we have a sidebar that will show us details about any selected elements.

<img width="800" src="https://user-images.githubusercontent.com/70295997/224855352-3a0d76b7-68cc-46cb-8c27-90213943bac1.png">

To select an element, all I need to do is start hovering over the screenshot. Appium Inspector will highlight sections it thinks contain an element for me. I'll go ahead and click the "Login Screen" list item here. When I do that, a bunch of information pops up on the right, and the corresponding XML node is selected in the source tree. Now let's have a look at the "Selected Element" section. There are three main parts to it here. Up top we have a series of buttons that allow me to take actions with this element. If I want, I could tap, send keys, or clear this element. I'm going to hold off on that for now.

<img width="800" src="https://user-images.githubusercontent.com/70295997/224862583-166d010b-f33e-4039-bb8a-94ed80bc6f00.png">

At the bottom of this section, we have a table with all the attributes this element contains, and their values. We can discover these with code using the <code>element.get_attribute()</code> method. Or we see the same name and label attributes we saw in the page source we printed out earlier. But it's what's above this part that's most interesting. Just below the action buttons is another table that has suggested locator strategies and selectors. This is pretty cool. Just by clicking an element in the screenshot, Appium Desktop's Inspector is able to suggest how I might find this element using an Appium locator strategy and selector. In some cases, multiple strategies might be available, as is the case here.

<img width="500" src="https://user-images.githubusercontent.com/70295997/224863602-4399e41a-753a-4401-831f-4b5cb0bb4d80.png">

I can either find this element by accessibility ID, using the 'Login Screen' string, or I can find it by XPath, using the query <code>//XCUIElementTypeOther[@name='Login Screen']</code>. Of course I'll choose Accessibility ID since it will be faster and potentially more stable. But notice that the XPath query Appium recommends is actually quite a good one. It's recommending a query which is using a unique attribute of the element, to make sure that it is as stable as possible. If the Inspector isn't able to find a good strategy and selector for you to use, it will always recommend an XPath query of some kind. But if this is not based on unique element attributes, a warning will be displayed so you know it might not be the best stable selector.

There's a lot more to the Inspector, and I hope you spend some good time playing around with it. For now, I just wanted to show you how to use the Inspector to have it recommend locator strategies and selectors for you. In most cases, this is enough to get you through automating your app. Just load up the Inspector, find the elements you'll need to interact with, and make a note of any good strategies and selectors it recommends. You can then implement them in your code. So that's all for now, but by all means, keep exploring the Inspector, and you might discover some fun features including action recording and code generation.

And, as always, don't forget to cleanly quit your session by clicking "Quit Session and Close Inspector".

<img width="400" src="https://user-images.githubusercontent.com/70295997/224864712-7076697f-77f3-4bc7-8a45-1eae62e88bc4.png">



    
