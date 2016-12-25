---
layout: post
title: 'web框架之行为驱动'
tags: [AutoTest]
description: >

---

**前言**

行为驱动开发英文：Behavior or drivern develoment（简称BDD），可以通过业务语言来编写自动化测试，让测试用例更加自然和简单，测试人员可以很清楚实现的逻辑。

BDD测试框架有 Ruby 中的 Cucumber，Python 中的 Behave，Lettuce 及 Freshen，.Net 中的 SpecFlow 等。而JAVA语言，使用的是cucumber的java版本框架。Cucumber是Ruby语言开发的，支持在jvm平台上使用各类语言

**正文**

以百度搜索为例：

- 首先下载Cucumber相关的jar包，包括Cucumber-core.jar(核心包)，Cucumber-java.jar(java语言必备包),Cucumber-html.jar（生成html），Cucumber-testng.jar（使用testng执行测试并生成报告），testng.jar,Gherkin(步骤定义时需要)

- 新建search.feature,在里面输入：
英文版：
```
Feature: 搜索
Background：在http://baidu.com搜索selenium

Scenario： 在百度上搜索selenium
Given 打开http://baidu.com
When  输入关键字: selenium
Then  搜索结果: selenium显示出来
```
中文版：
```
功能: 搜索
背景：在http://baidu.com搜索selenium

场景： 在百度上搜索selenium
假如 打开http://baidu.com
当  输入关键字: selenium
那么  搜索结果: selenium显示出来
```

其中Feature，Background，Scenario，Given，When，Then都是关键字， 也可以用中文代替

- 新建SearchStepdefs.java，将上面的步骤解析，主要解析Given，When，Then三个关键字

```
public class SearchStepdefs {
    private Search search = null;
    private String result = null;
    @Given("^打开http://www.so.com/$") 
    public void open(){
    search = new Search();
    search.open();
 }
    @When("^输入关键字：(.*)$")
    public void input(String keyword){
    result = search.find(keyword);
 }
    @Then("^搜索结果：(.*)显示出来$")
    public void result(String expectedResult){
    Assert.assertEquals((Object)expectedResult,(Object)result);
}
}
```
- 新建Search.java,以便上面的调用：

```
public class Search {
	public WebDriver driver;
	public String baseUrl;

	//初始化浏览器操作
	public void setUp() {
		driver = new FirefoxDriver();
		baseUrl = "http://www.so.com/";
		driver.manage().timeouts().implicitlyWait(30, TimeUnit.SECONDS);
		driver.manage().window().maximize();
	}

	//打开url
	public void open() {
		setUp();
		driver.get(baseUrl);
	}

	//查找cucumber是不是显示了
	public String find(String keyword) {
		String result = null;
		WebElement element = null;

		driver.findElement(By.name("q")).clear();
		driver.findElement(By.name("q")).sendKeys(keyword);
		driver.findElement(By.id("search-button")).click();
		driver.manage().timeouts().implicitlyWait(30, TimeUnit.SECONDS);

		element = driver.findElement(By.id("first"));
		if (element.getText().contains("cucumber")) {
			result = "cucumber";
		}
		quit();
		return result;
	}

	//释放资源
	public void quit() {
		driver.quit();
	}
}
```
- 新建TestRunnerWithTestNG.java,指定testNG中feature文件和生成报告的路径

```
    @CucumberOptions(features="classpath:cucumber/resources",plugin=
{"pretty", "html:result/cucumber/cucumber-html-report","json:result/cucumber/cucumber-report.json"})
    public class TestRunnerWithTestNG extends AbstractTestNGCucumberTests{

}
```

- 最后执行testng.xml
