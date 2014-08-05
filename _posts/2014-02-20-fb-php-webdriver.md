
---
layout: post
title: facebook / php-webdriver
categoty: webdriver, php
---

This article will explain how to do Webdriver tests using the Facebook Webdriver wrapper and PHPUnit. The advantages of using Webdriver over the previous version of Selenium is that Selenium 2 Webdriver uses an API to run tests in the browser rather than using Javascript with an iframe as in version 1. This allows new types of tests which were not previously possible. Version 2 may also be more stable across browsers and platforms although I've not tested this one yet.

Another author has created instructions here:
[Using Selenium 2 & PHPUnit to automate browser testing](http://edvanbeinum.com/using-selenium-2-phpunit-to-automate-browser-testing)

Before beginning ensure you have a working installation of Apache, PHP, Pear and PHPUnit on Windows. Here is my post on this subject:For Dummies Guide to Installing Pear, PHPUnit...

You may also want to update PHPUnit if it is an old version by opening the command line and typing:
-----------------------------------
  pear upgrade phpunit/PHPUnit

-----------------------------------

Ensure Curl is installed on your server computer, without this you will get an error, 
-----------------------------------
  Fatal error: Call to a member function close() on a non-object
  
-----------------------------------


To switch Curl on, go to your __php.ini__ file and ensure that __extension=php_curl.dll__ is uncommented. Restart the webserver.

Download: Facebook Selenium Webdriver library from: [Facebook / PHP-Webdriver](https://github.com/facebook/php-webdriver) . Unpack this and place it into a directory where you want to run your tests.

Download the latest Selenium 2 Server Standalone from the [Selenium website](http://seleniumhq.org/download/).


Open a new php document and put the following code in it: 
------------------------------------
  <?php
  require_once '../php-webdriver/__init__.php';
  
  class WebdriverTest extends PHPUnit_Framework_TestCase
  {
      /** 
      * @var WebDriverSession
      */
      protected $_session;
  
      public function setUp()
      {
          parent::setUp();
          $web_driver = new WebDriver();
          $this->_session = $web_driver->session();
      }
  
      public function tearDown()
      {
          $this->_session->close();
          unset($this->_session);
          parent::tearDown();
      }
  
      public function test_mytest()
      {
          $this->_session->open('https://github.com');
  
          $this->_session->element(
              'id',
              'pics'
          )->click();
          
          echo $this->_session->element('id','pics')->text();
  
          $this->assertSame(
              'https://startingpage.com/', 
              $this->_session->url()
          );
      }
  }
  
-------------------------------------------------

Save it as __WebdriverTest.php__. The script will open __https://github.com__, click on the images button and then assert that the page is now "https://startingpage.com/". Go to the link at the start of the article for a fuller explanation of this code. 
Running the Test

Open command prompt > cd to the directory containing the standalone server and run it with:

-------------------------------------------
  java -jar selenium-server-standalone.jar
  
-------------------------------------------

###Run the test using:
--------------------------------------------

  phpunit --log-junit output.xml HomePageTest.php
  
  ------------------------------------------

This will run the script and place any output including assert/verify functions into the __output.xml__ file. The output from this test should be:

--------------------------------------------
  <?xml version="1.0" encoding="UTF-8"?>
  <testsuites>
    <testsuite name="WebdriverTest" file="C:\_projects\s2\WebdriverTest.php" tests="1" assertions="1" failures="0" errors="0" time="10.017330">
      <testcase name="test_mytest" class="WebdriverTest" file="C:\_projects\s2\WebdriverTest.php" line="25" assertions="1" time="10.017330"/>
    </testsuite>
  </testsuites>
  
-------------------------------------------

Unless they have changed something on the website. 

###Learning More

The Facebook PHP Webdriver wrapper is based on the JSON Wire Protocol so many of the functions have the same name or are similar. You can look up more information by going to the protocol's function page at:The WebDriver Wire Protocol.

 __http://code.google.com/p/selenium/wiki/JsonWireProtocol__

You can learn more about the Facebook Webdriver wrapper at: 

[Readme](https://github.com/facebook/php-webdriver#readme)
[Examples page](https://github.com/facebook/php-webdriver/wiki/Example-command-reference)
[Running different browsers](https://github.com/facebook/php-webdriver/wiki/Launching-Browsers)
