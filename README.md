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



## Element discovery: Appium inspector
