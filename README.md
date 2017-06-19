# Goal
Our goal is to get together Selenium best practices. Feedback is welcome. If you see anything wrong or know better way open an issue or create a pull request, just fork it and change it. We are open to learn and hear new ideas. 

# Selenium-best-practices
Selenium is a portable software testing framework for web applications. It also provides a test domain-specific language to write tests in a number of popular programming languages, including Java, C#, Groovy, Perl, PHP, Python and Ruby.

* [General](#general)
  * [Use PageObjects Pattern](#use-pageobjects-pattern)
  * [Prefered Selector Order](#prefered-selector-order)
  * [Create Ordered Tests](#create-ordered-tests)
  * [Get Name of the Running Test](#get-name-of-the-running-test)
  * [Do not Use Magic String](#do-not-use-magic-strings)
  * [Behavior Driven Design](#behavior-driven-design)
  * [Use Build Automation System](#use-build-automation-system)
  * [Learn Continuous Integration Tools](#learn-continuous-integration-tools)




## Use PageObjects Pattern

Page Object is a Design Pattern which has become popular in test automation for enhancing test maintenance and reducing code duplication. An implementation of the page object model can be achieved by separating the abstraction of the test object and the test scripts.

**Advantages of using Page Object Pattern:**
* Easy to Maintain
* Easy Readability of scripts
* Reduce or Eliminate duplicacy
* Re-usability of code
* Reliability
* There is a clean separation between test code and page specific code such as locators  and layout.

Use multi config for each environment. Make it changeable. Read your test case value from config. Don't use a static.

* StageConfig
 * loginUrl=http://stage.com
 * validUsername=testUser@test.com
 * validPassword=123456
 
* DevConfig
 * loginUrl=http://dev.com
 * validUsername=testUser@test.com
 * validPassword=123456

**Shortcut**
* Don't use magic strings.
* Separate to WebPageElement, WebPageObject, WebTest.




## Prefered Selector Order

Preferred selector order : id > name >links text> css > xpath

CSS and XPath are location based selectors, they are slower than other selectors.

* Id and name are often the easiest and sure way.
* CSS = Id + name.
* Last solution should be xpath.


## Create Ordered Tests

```java
import org.junit.runners.MethodSorters;

//Running tests in order of method names in ascending order
@FixMethodOrder(MethodSorters.NAME_ASCENDING)
//Running tests in order of method names in ascending order
@FixMethodOrder(MethodSorters.JVM)
```


## Get Name of the Running Test

```java
import org.junit.rules.TestName;

@Rule
public TestName name = new TestName();

//example usage; writes every test name before it runs.
@Before
    public void logBeforeTestsStart() {
        log.info(name.getMethodName());
    }

```


## Do not use Magic Strings

It improves readability of the code and it's easier to maintain.

```java
WebDriver driver = new HtmlUnitDriver();
// Don't use, create Constants each element name
WebElement element = driver.findElement(By.name("sample"));
driver.quit();
```


## Behavior Driven Design

BDD is principally an idea about how software development should be managed by both business interests and technical insight, the practice of BDD assumes the use of specialized software tools to support the development process.
Think all test scenarios.

Example BDD 

```
Feature: Sign up

Sign up should be quick.
Write all steps.
1- Open Browser
2-Navigate to Url
3-Click Sign Up Button
4-Write Valid Username and Password
5-Click Register.

Scenario: Successful sign up
New users should get a confirmation email and be greeted
personally by the site once signed in.

Given I have chosen to sign up
When I sign up with valid details
Then I should receive a confirmation email
And I should see a personalized greeting message

Scenario: Duplicate email

Where someone tries to create an account for an email address
that already exists.

Given I have chosen to sign up
But I enter an email address that has already registered
Then I should be told that the email is already registered
And I should be offered the option to recover my password

```


###Simple Example


##Page Element Class
```java 
//Write clear class  name.
public class WebLoginPageElement {

    private String emailInputId;
    private String passwordInputId;
    public WebLoginPageElement() {
        this.emailInputId = "Email";
        this.passwordInputId = "Password";
        }
        
    public String getEmailInputId() {
        return emailInputId;}
    
    public String getPasswordInputId() {
        return passwordInputId;}
        
```

##Page Object Class

```java
public class WebLoginPage extends Function {

   //Make own custom WebElement Class.
   private WebElementFactory elementFactory;
   private IWebAction elementAction;
   private WebLoginPageElement loginPageElement;

   public WebLoginPage() {
     elementFactory = new WebElementFactory(driver);
     elementAction = new WebAction(driver);
     loginPageElement = new WebLoginPageElement();
    }


   public WebLoginPage login(String email, String password) {
    
    WebElement emailInputElement = 
      elementFactory.createElementById(loginPageElement.getEmailInputId());
      elementAction.clear(emailInputElement);
      elementAction.sendKeys(emailInputElement, email);

    WebElement passwordInputElement = 
      elementFactory.createElementById(loginPageElement.getPasswordInputId());
      elementAction.clear(passwordInputElement);
      elementAction.sendKeys(passwordInputElement, password);

    WebElement loginSubmitButton =
      elementFactory.createElementByCssSelector(loginPageElement.getSubmitBtnCss());
      elementAction.click(loginSubmitButton);
        return this;}
```


##Test Class

```java

@FixMethodOrder(MethodSorters.NAME_ASCENDING)
public class WebLoginTest extends Function {

    private static org.apache.log4j.Logger log = Logger.getLogger(WebLoginTest.class);
    WebElementFactory elementFactory = new WebElementFactory(driver);
    private static WebLoginPageElement loginPageElement = new WebLoginPageElement();
    
    @Rule
    public TestName name = new TestName();

    @BeforeClass
    public static void setUpBeforeClass() throws Exception {
        PropertyConfigurator.configure(IConstants.LOG4j_PROP);
        openBrowser(RunAllWebTests.getConfigProperties().getLoginUrl());
    }
       @AfterClass
    public static void tearDownAfterClass() throws Exception {
        driver.close();
        driver.quit();
    }
    
    //Test all scenarios. 
    @Test
    public void loginWithEmptyUsername() throws Exception {
        //Call loginPage 
        new WebLoginPage().login(
                RunAllWebTests.getConfigProperties().getEmptyUsername(),
                RunAllWebTests.getConfigProperties().getValidPassword())
                .assertTruePageContain(loginPageElement.getEmptyUsernameAssert());
    }
    
    @Test
    public void loginWithWrongFormat(){
    //...
    }
    
    @Test
    public void loginWithWrongFormatId(){
    //...
    }
```

