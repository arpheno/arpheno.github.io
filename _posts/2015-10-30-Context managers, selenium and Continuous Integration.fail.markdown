---
layout: post
title:  Context managers, selenium and Continuous Integration
date:   2015-10-30 09:43:35
categories: backendfail selenium circleci
---

I was having huge problems getting my selenium tests to run on CircleCI.

I tried to be really smart and launch an `xvfb/selenium/webdriver` docker container on their infrastructure which
would then be used to run my selenium tests. This worked beautifully in a local Vagrant VM, but would fail for some 
reason on the CircleCI infrastructure.

**It turns out I was trying to be too clever for CircleCI. **

They run an `xvfb` screen by default on port `:99`, which is why my selenium node would silently fail.

To get my tests running i had to replace

    @given('I have a web browser')
    def browser():
        return webdriver.Remote(
            command_executor='http://127.0.0.1:4444/wd/hub',
            desired_capabilities=DesiredCapabilities.CHROME)
with:

    @given('I have a web browser')
    def browser():
        try:
            return webdriver.Remote(
                command_executor='http://127.0.0.1:4444/wd/hub',
                desired_capabilities=DesiredCapabilities.CHROME)
        except:
            return webdriver.Chrome()

This setup runs nicely on my local vagrant, and on CircleCI.

---

**Context managers are awesome:**

I used to have lots of content functions like this:

    def launch_celery():
        local('cd backendfail && celery -A settings worker --loglevel=INFO &')  # this is probably really bad
    def kill_celery():
        local('killall celery')
Which I used like this:

    def coverage():
        launch_celery()
        launch_runserver()
        launch_selenium()
        local('coverage -m py.test backendfail/tests')
        kill_celery()
        kill_runserver()
        kill_selenium()

This was ugly because when `coverage` failed, `celery` wouldnt be killed and ruin my further tests.

Context managers are generators used with the `with` statement:

    @contextmanager
    def celery():
        local('cd backendfail && celery -A settings worker --loglevel=INFO &')  # this is much better
        yield
        local('killall celery')  # this is still pretty bad
The `yield` statement passes control to the inner block of a `with` statement.

With context managers I can now do:

    def coverage():
        with celery(), runserver(), selenium():
            local( r'coverage run -m py.test backendfail/tests')

This saves a lot of typing definitions and cleans up automatically, even if `coverage` fails.

Awesome.
