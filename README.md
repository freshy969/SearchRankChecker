# Search Rank Checker

## Description

Search Rank Checker is an Asp.Net Core MVC based application designed to do google search for checking the rank of any given url for the search keywords provided.

## Requirements

User should be able to enter the desired url and the keywords in the given UI. Clicking search will get the search results from Google and calculate the rating or position in the search. It will then be shown as a string on the UI.

## Programming Language / Technology Used

* .Net Core 3.1
* Asp.Net Core MVC 3.1
* C#
* NUnit
* Moq

## UI

![Search Rank Checker UI](./Assets/SearchRankChecker_UI.png "Search Rank Checker UI")

![Search](./Assets/SearchRankChecker_Search.png "Search")

![Search Results As Rank String](./Assets/SearchRankChecker_RankString.png "Search Results As Rank String")

## Crawling Google Search

When we started to hit google search initially via HttpClient, it gave us a sort of transormed html where all the class names were encoded and the html structure was bit different. Then we figured out that we should be sending a **User-Agent** to tell google we are a browser. Once we did that, we started to get the html structure similar to what we were getting when directly using Google Chrome.

Next problem was to figure out how to extract the required data out of the raw string. After some searching, we found a very good regex to match the results within the search result's raw string.

After getting the desired search results, next was to find the url user provided via the UI and store the position of the match.

Next we just return the results to the client.

## Some Challenges

### Data Disappear On Posting Form

If we had more time, we could have probably replaced the partial view with **ViewComponent** so it can have its own class and logic like having default values and retaining form data after POST. Also, result string for showing rank could also be passed via the view model.

### Google Search and Region

Even when we are using http://google.com.au as the search engine url, the results returned by Google Search is New Zealand based. We are currently resolving this issue by adding australia at the end of the search keywords as seen in the attached image.

We attempted to override the Culture and UI Culture of the default thread by adding the dropdown on the UI (See Screenshot) and the code below:

```csharp
[HttpPost]
public IActionResult SetLanguage(SearchViewModel searchViewModel)
{
    // Set Culture
    CultureInfo newCulture = new CultureInfo(searchViewModel.SearchRegion);

    CultureInfo.DefaultThreadCurrentCulture = newCulture;
    CultureInfo.DefaultThreadCurrentUICulture = newCulture;

    return RedirectToAction(nameof(Index), new SearchViewModel
    {
        SearchRegion = Thread.CurrentThread.CurrentCulture.Name
    });
}
```

The region gets changed successfully but still no luck with the google search. We tried similar thing at other places in the code like when setting up the HttpClient or HttpRequestMessage.

We also tried sending in Accept-Language header with the request but again no luck.

So the only thing works is the word **"australia"**.

### Different Search Results from Browser and Code

Searching for same URL and Keywords, we get only one result in the top 100 from the browser. Whereas, if we search through our code, we get two results. On debugging we found out it gets different directories when searched through our application code:

![Coverage Report](./Assets/SearchResultsThroughCode.png "Coverage Report")

## Test Coverage

![Coverage Report](./Assets/CoverageReport.png "Coverage Report")

Coverage can definitly be improved if we have more time.

## Configuration Options

### Multiple Client Support for future versions

#### HttpClients

Multiple clients can be added via appSettings.json's **HttpClients** config entry. We only have Google Client as of now. (Note: Code will also require changes like new implementation of **ICrawlerService**)

#### LookupRegex

**LookupRegex** is used to match and extract out all the urls from the search results (10, 100 etc)

#### SearchDefaults -> DirectoryPath

**DirectoryPath** is used to match the exact url from the urls received through search

#### MaxSearchResults

MaxSearchResults is used to tell Google Search how many results we want to see on one page or in one go. Google supports 10, 20, 30, 40, 50 and 100. So we should set **MaxSearchResults** config to any of these or else you will get the default 10 results

## Author

Abul Hasan Lakhani

## Date

18th Feb 2020
