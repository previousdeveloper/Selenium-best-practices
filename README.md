# Selenium-best-practices
Selenium is a portable software testing framework for web applications.It also provides a test domain-specific language to write tests in a number of popular programming languages, including Java, C#, Groovy, Perl, PHP, Python and Ruby.

* [General](#general)
  * [Use PageObjects pattern](#use-pageobjects-pattern)
  * [Prefered selector order](#prefered-selector-order)
  * [Do not use Magic String](#do-not-use-magic-string)
  * [Behavior Driven Design](#behaviour-driven-design)
  * [Use build automation system](#use-build-automation-system)



### Use PageObjects pattern

Page Object is a Design Pattern which has become popular in test automation for enhancing test maintenance and reducing code duplication.An implementation of the page object model can be achieved by separating the abstraction of the test object and the test scripts.

**Advantages of using Page Object Pattern:**
* Easy to Maintain
* Easy Readability of scripts
* Reduce or Eliminate duplicacy
* Re-usability of code
* Reliability
* There is a clean separation between test code and page specific code such as locators  and layout.

Use multi config for each environment. Make it changeable.Read your test case value from config. Don't use a static.

* StageConfig
 * loginUrl=http://stage.com
 * validUsername=testUser@test.com
 * validPassword=123456
 
* DevConfig
 * loginUrl=http://dev.com
 * validUsername=testUser@test.com
 * validPassword=123456

**Shortcut**
* Don't use magic string.
* Separate to WebPageElement, WebPageObject,WebTest.


##Simple  example


###PAGE ELEMENT CLASS###
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

###PAGE OBJECT CLASS###
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

###TEST CLASS###
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

### Prefered selector order

Preferred selector order : id > name >links text> css > xpath

Css and Xpath are located based selector, and they are slower than other selectors.

* Id and name are often the easiest and sure way.
* Css = Id + name.
* Last solution should be xpath.



### Do not use Magic String

It improves readability of the code and it's easier to maintain.

```java


WebDriver driver = new HtmlUnitDriver();
// Don't use, create Constants each element name
WebElement element = driver.findElement(By.name("sample"));
driver.quit();


```
