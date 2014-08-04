---
layout: post
title: Behat - MinkExtension
category: bdd
---

*If you just want the code, you can find it [over on Github](https://github.com/tikolakin/behat-minkextension)*

Around a week ago, I wrote up my experience using Behat, Mink and Selenium2/WebDriver. As it turns out, whilst it was a good learning experience and it did work, I was going about it completely the wrong way.

Whilst working with Mink, I kept seeing mentions to MinkExtension, but didn’t see any details anywhere so I just ignored it. Soon after publishing my last post, [Everzet](https://twitter.com/everzet) commented and posted a link to some MinkExtension docs. I’ve run through the install process and it’s much easier. However, the docs aren’t complete for using Selenium2 so I’m still going to document the process here.

###Installing with Composer

As before, the easiest way to get Behat installed is to use [Composer](http://getcomposer.org/). Once you’ve [installed Composer](http://getcomposer.org/doc/01-basic-usage.md), create a **composer.json** file that contains the following:

---------------------------------------
		{
	    "require": {    
	            "behat/behat":                  ">=2.2.2",
	            "behat/mink":                   "1.4@stable",
	            "behat/mink-selenium2-driver": "*",
	            "behat/mink-extension":         "*"
	    }
	}

---------------------------------------

###Initialising Behat

Once that completes, you should be able to run behat with **vendor/behat/behat/bin/behat**. That gets old pretty quickly though, so I like to run **ln -s vendor/behat/behat/bin/behat behat** to create an alias in the current directory. If you do the same, you’ll be able to run behat just by typing **./behat**

You should be ready to run **./behat --init** and get your project started. This will create the features folder, along with **FeatureContext.php**. You’ll also need a **behat.yml** file. Create this file in the same directory as **composer.json** with the following content

---------------------------------------
	default:
	  extensions:
	    Behat\MinkExtension\Extension:
	      base_url:  'http://YOUR-DOMAIN.com'
	      default_session: selenium2
	      browser_name: 'firefox'
	      selenium2:                    
	        capabilities: { "browser": "firefox", "version": "14"}

---------------------------------------

The important bits here that aren’t documented in MinkExtension are that your default_session should be selenium2, and that instead of using **selenium2: ~**, you need to use it as a section and pass in your capabilities. Without this, MinkExtension will try and use Firefox version 8 and fail.

###Creating a subcontext

Like before, I prefer to set up subcontexts to keep things clean, so create a new file in **features/bootstrap** named **GuiContext.php** with the following content:

---------------------------------------
	use Behat\Gherkin\Node\PyStringNode;

	use Behat\Mink\Mink,
	    Behat\Mink\Session,
	    Behat\Mink\Driver\Selenium2Driver,
	    Behat\MinkExtension\Context\MinkContext;

	use Selenium\Client as SeleniumClient;

	require_once 'PHPUnit/Autoload.php';
	require_once 'PHPUnit/Framework/Assert/Functions.php';

	class GuiContext extends MinkContext
	{

	}

---------------------------------------

By extending MinkContext, we inherit a lot of built in steps to use when driving Selenium. First though, we need to edit **features/bootstrap/FeatureContext.php** to register our new subcontext. Add the following code inside the __construct method:

---------------------------------------
	$this->useContext('gui',
	    new GuiContext($parameters)
	);

---------------------------------------

You’ll be able to run **./behat -di** now to view a list of all of the available steps, all provided by MinkExtension.

###Selenium Server

Now, I’m not sure how this would work for people that work on GUI machines, but I develop in a virtual machine so there’s no way to run browsers on them. To get around this, I use [Selenium Grid](http://code.google.com/p/selenium/wiki/Grid2) to run the tests remotely on my iMac.

Firstly, you’ll want to download [Selenium Server](http://seleniumhq.org/download/) on all the machines involved.

You’ll want to run the following command on the machine that’s running the tests (the machine on which you run **./behat**). This sets up a hub that can receive connections from a node reporting that there are browsers available to test with.

---------------------------------------
	java -jar selenium-server-standalone-2.25.0.jar -role hub

---------------------------------------
On the machine that you want to run the browser on, run the command:

---------------------------------------
	java -jar selenium-server-standalone-2.25.0.jar -role node -hub http://192.168.56.101:4444/grid/register -browser browserName=firefox,version=14,maxInstances=1

---------------------------------------

This starts up Selenium server in node mode. Node mode registers that you have available browsers and fires up browser windows to automate.

The IP address provided should be the IP address of the machine that you set up as a hub. The browser parameter is what browser will be used to run the test. Finally, I set maxInstances to 1 so that we only run one test at a time and my machine doesn’t fall over.

###Your first feature

Just like last time, let’s build a simple feature to show how Mink + Behat work together. This time though, I’m going to use only the built in steps. I won’t write any myself. So, create **features/google.feature** and give it the following content:

--------------------------------------
	Feature: Visit Google and search

	Scenario: Run a search for Behat

--------------------------------------


You might remember that one of the steps listed when we ran **./behat -di** was

--------------------------------------
	Given /^(?:|I )am on "(?P<page>[^"]+)"$/.

--------------------------------------
We’re going to use this to visit google.com. Your **google.feature** file should look like this:

--------------------------------------
	Feature: Visit Google and search

	Scenario: Run a search for Behat
    	Given I am on "http://google.com/?complete=0"

--------------------------------------

The next thing to do is to fill in the search box. We’re going to use the step

---------------------------------------
	When /^(?:|I )fill in "(?P<field>(?:[^"]|\\")*)" with "(?P<value>(?:[^"]|\\")*)"$/ 

---------------------------------------

to achieve this. Then, we’re going to use the step 

---------------------------------------
	When /^(?:|I )press "(?P<button>(?:[^"]|\\")*)"$/

---------------------------------------

to click on the search button. Once you add those, your feature file should look something like the following:

---------------------------------------
	Feature: Visit Google and search

	Scenario: Run a search for Behat
	    Given I am on "http://google.com/?complete=0"
	    When I fill in "lst-ib" with "Behat"
	    And I press "Google Search"

---------------------------------------

Finally, we want to ensure that the first result is “Behat — BDD for PHP”. To do this, we use the step 

---------------------------------------
	^(?:|I )should see "(?P<text>(?:[^"]|\\")*)" in the "(?P<element>[^"]*)" element$/

---------------------------------------
	Feature: Visit Google and search

	Scenario: Run a search for Behat
	    Given I am on "http://google.com/?complete=0"
	    When I fill in "lst-ib" with "Behat"
	    And I press "Google Search"
	    Then I should see "Behat — BDD for PHP" in the ".vsc:first-child a" element

----------------------------------------

This looks as though it should work, but the final query runs before the search has time to execute, so we’re going to have to introduce a delay. There was no method listed when we ran **./behat -di** to pause for a second or two, so we’ll add one to **GuiContext.php** (I know I said we wouldn’t add any, but this is barely a method).

So, if you add the following step to GuiContext.php:

-----------------------------------------
	/**
	 * @When /^wait (\d+) seconds?$/
	 */
	public function waitSeconds($seconds)
	{
	    $this->getSession()->wait(1000*$seconds);
	}

------------------------------------------

Our feature file becomes:

----------------------------------------
	Feature: Visit Google and search

	Scenario: Run a search for Behat
	    Given I am on "http://google.com/?complete=0"
	    When I fill in "lst-ib" with "Behat"
	    And I press "Google Search"
	    And wait 1 second
	    Then I should see "Behat — BDD for PHP" in the ".vsc:first-child a" element

----------------------------------------

If we run **./behat** all the steps should go green. Congratulations, you have a passing test that uses (almost) exclusively built in MinkExtension steps. This is much easier than integrating Mink yourself, and it even fixes the issue with the browser window appearing that my previous solution had.