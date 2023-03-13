# Determining Mobile Element Selectors in Appium

*Just as with websites, tools exist to inspect native apps. We can use these tools, of which Appium Inspector is our favorite example, to help find out which selectors we can use to find specific elements for our automation code.*

## Element discovery: Getting app source

Let's learn how to decide on locator strategies and selectors for finding elements in our mobile apps. There are two basic ways to do this. I'm going to show you one way first, even though the second way is much better and easier. So what's the first way, and why should we care about it? The first way to discover locators is to simply print out the page source and examine it yourself, learning from the various element properties what you need to know in order to find a given element.

This is an important skill to learn because it's one you may need to fall back on at some point. In certain circumstances or environments you might not have access to the tools that make your life easier in determining selectors. But if you can run an Appium session, you can get the page source! In fact, examining the page source is a useful debugging technique, not just a technique for finding elements.

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

What we want to do is print out the page source instead:

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

Once this is complete, what do we see? The script has finished, having printed out a bunch of stuff here. This is an XML document representing the structure of the UI at the time we requested the page source. Every element that Appium knows about is represented as an XML node here. So if you don't see an element in this document, then Appium will not be able to find it as a normal UI element. Let's look down through this for a bit. I see that each element has a type, like <code>XCUIElementTypeStaticText<code> or <code>XCUIElementTypeOther</code>. There's not too many different types, really, so they're not necessarily all that helpful. But each element has a set of attributes, which can be pretty helpful. For example, I see this section here which appears to show the list of views in my app. Looking at the <code>name</code> attribute, I see "Echo Box", "Login Screen", and so on. And this <code>name</code> attribute is actually quite important, since on iOS, if an element has a <code>name</code> attribute, that means we can find that element using the Accessibility ID strategy, and for our selector using the value of this <code>name</code> attribute. So looking at this source, I could find any of these elements like Echo Box or Login Screen simply by using the Accessibility ID strategy.

As you can imagine, looking at the XML output here is also useful for coming up with XPath selectors, because when you run an XPath query to find an element, it is precisely this document which is consulted in order to do so! So once you've learned a bit more advanced XPath, you can walk through this document and figure out what a good XPath query might be for a particular element in the document that you want to find with Appium.
    
    


## Element discovery: [Appium inspector](https://youtu.be/PG0PiHnukXA)

...


    
