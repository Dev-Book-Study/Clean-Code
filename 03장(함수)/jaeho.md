## 3장 함수(Function)
---
<details markdown="1">
  <summary>목록 3-1 HtmlUtil.java (FitNesse 20070619)</summary>

  ```java
  public static String testableHtml(
    PageData pageData;
    boolean includeSuiteSetup
  ) throws Exception {
    WikiPage wikiPage = pageData.getWikiPage();
    StringBuffer buffer = new String Buffer();
    if (pageData.hasAttribute("Test")) {
      if (includeSuiteSetup) {
        WikiPage suiteSetup = PageCrawlerImpl.getInheritedPage(
          SuiteResponder.SUITE_SETUP_NAME, wikiPage
        );
        if (suiteSetup != null) {
          WikiPagePath pagePath = suiteSetup.getPageCrawler().getFullPath(suiteSetup);
          String pagePathName = PathParser.render(pagePath);
          buffer.append("!include - setup .")
                .append(pagePathName)
                .append("\n");
        }
      }
      WikiPage setup = PageCrawlerImpl.getInheritedPage("SetUp", wikiPage);
      if (setup != null) {
        WikiPagePath = setupPath = wikiPage.getPageCrawler().getFullPath(setup);
        String setupPathName = PathParser.render(setupPath);
        buffer.append("!include - setup .")
              .append(setupPathName)
              .append("\n")
      }
    }
    buffer.append(pageDate.getContent());
    if (pageData.hasAttribute("Test")) {
      WikiPage teardown = PageCrawlerImpl.getInheritedPage("Teardown", wikiPage);
      if (teardown != null) {
        WikiPage.getPageCrawler().getFullPath(teardown);
        String tearDownPathName = PathParser.render(tearDownPath);
        buffer.append("\n")
              .append("!include -teardown .")
              .append(tearDownPathName)
              .append("\n")
      }
      if (includeSuiteSetup) {
        WikiPage suiteTeardown = PageCrawlerImpl.getInheritedPage(
          SuiteResponder.SUITE_TEARDOWN_NAME,
          wikiPage
        );
        if (suiteTeardown != null) {
          WikiPagePath pagePath = suiteTeardown.getPageCrawler().getFullPath (suiteTeardown);
          String pagePathName = PathParser.render(pagePath);
          buffer.append("!include - teardown .")
                .append(pagePathName)
                .append("\n");
        }
      }
    }
    pageData.setContent(buffer.toString());
    return pageData.getHtml();
  }


  ```
> 이 코드의 문제는 추상화 수준이 높을 뿐만 아니라, 긴 코드, 여러겹의 if문, 이상한 문자열 등으로 코드를 이해하기 어렵다..

</details>