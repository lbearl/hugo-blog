---
title: "PageRuleAdmin - Add CloudFlare Forwarding URLs easily"
date: "2018-01-02"
categories: 
  - "software-development"
---

For a lot of the things my wife and I work on it is useful to have redirects from one page to another. In the classic developer spirit, instead of always logging into the CloudFlare dashboard and manually updating everything, I threw together a small utility in a few hours which allows her (a relatively non-technical user) to easily administrate the forwarding url page rules for all of the domains that we have running in CloudFlare (which is most of them).

The application is just a basic ASP.NET Core 2.0 web application using traditional MVC patterns. CloudFlare, unfortunately, doesn't really have a nice C# bindings for their SDK, so I just implemented the necessary calls using `HTTPClient()`. The core of the logic for interacting with the API is:

```
private async Task PerformWebRequest(string uri, HttpMethod method, string postBody = null)
{
var client = new HttpClient();

var request = new HttpRequestMessage
{
RequestUri = new System.Uri(uri),
Method = method,
};

if(postBody != null)
{
request.Content = new StringContent(postBody, Encoding.UTF8, "application/json");
}

request.Headers.Add("X-Auth-Email", UserEmail);
request.Headers.Add("X-Auth-Key", ApiKey);

var result = await client.SendAsync(request);

var resultString = await result.Content.ReadAsStringAsync();

return JsonConvert.DeserializeObject(resultString);
}
```

Note that the function is generic as I wanted to centralize all of logic which actually calls out to the API.

Take a look [here](https://github.com/lbearl/PageRuleAdmin) for the github. I host the code on a Hyper-V Ubuntu Server VM, so unfortunately there isn't a running example available.

Let me know if you find a use for it!
