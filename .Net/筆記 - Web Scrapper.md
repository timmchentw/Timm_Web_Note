# Web Scrapper

## Selenium

### 網址輸入

```C#
driver.Navigate().GoToUrl("Website URL");
```

### 選取DOM

```C#
driver.FindElement(By.Id("target-id"));
driver.FindElement(By.Name("target-name"));
driver.FindElement(By.ClassName("target-class"));
driver.FindElement(By.TagName("target-tag-name"));
driver.FindElement(By.XPath("//xpath"));
driver.FindElement(By.LinkText("target-text"));
//...
```

### 雙擊

```C#
var actions = new OpenQA.Selenium.Interactions.Actions(driver);
actions.DoubleClick(driver.FindElement(By.Id("target-id"))).Perform();
```

### 更改Select

```C#
var dropDown = new SelectElement(driver.FindElement(By.Id("target-id")));
dropDown.SelectByText("option-text");
dropDown.SelectByValue("option-value");
dropDown.SelectByIndex("option-index");
```

### 點選Popup Alert

```C#
var alert = driver.SwitchTo().Alert();
alert.Accept();
```

### 指定Iframe

```C#
driver.SwitchTo().Frame(driver.FindElement(By.Id("target-id")))
```

## XPath

### Search by Tag

```XPATH
//input[@class='target-class']
//image[contains(@class, 'target-class')]
```

### Search by Id

```XPATH
//*[@id='target-id']
//*[contains(@id, 'target-id')]
```

### Search by Name

```XPATH
//*[@name='target-name']
//*[contains(@name, 'target-name')]
```

### Search by Class Name

```XPATH
//*[@class='target-class']
//*[contains(@class, 'target-class')]
```

### Search by Text

```XPATH
//*[text()='target-text']
//*[contains(text(), 'target-text')]
```

### Search hierarchy

```XPATH
//table[@id='target-id']/tr[0]/td[0]
```
