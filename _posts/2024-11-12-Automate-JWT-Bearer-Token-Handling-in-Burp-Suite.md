---
title: Automating JWT Bearer Token Handling in Burp Suite
date: 2024-11-12 08:20:00 +/-TTTT
categories: [web,proxy]
tags: [web,proxy,burp,burpsuite,configuration]     
description: A step-by-step guide on configuring Burp Suite to automatically handle JWT authorization tokens.
pin: true 
toc: true
comments: false
mermaid: false
---


# Introduction

While performing security testing on web applications, we often encounter challenges that test our patience and skills. As powerful as Burp Suite is, configuring it to handle certain scenarios isn't always straightforward. One common issue is dealing with JWT (JSON Web Tokens) or Authorization tokens that expire frequently. How do you test an application when the access token expires every 180 seconds (3 minutes)? Most online resources focus on cookie session handling, which doesn't help with JWTs.

To save you time and keep your sanity, I've written this guide on how to set up Burp Suite to handle JWT/Authorization tokens automatically.



### The Problem

I could not find a native way in Burp Suite to update JWT tokens, as its session handling rules utilize a cookie jar that only manages session cookies and does not support headers. Manually repeating this process every few minutes is tedious and inefficient. Thankfully, there's an extension that addresses this issue.


### Configuring Burp Suite

We'll configure Burp Suite to automatically:

1. Detect when the JWT token has expired.
2. Fetch a new token.
3. Update the Authorization header with the new token.

### Step-by-Step Guide

1. **Install the "Add Custom Header" Burp Extension from BApp Store.**

2. **Create a Macro to Fetch a New Token**

   - From the **Proxy** tab, go to **Proxy Settings** > **Sessions** > **Macros**.
   - Click **Add** to create a new macro.
   - In the **Macro Recorder**, double-click on the "Filter settings: ..." which will launch the HTTP history filter.
   - Find the request that obtains the JWT token and click **OK**.
   - In the **Macro Editor**, under **Macro Items**, select the request and click **Configure Item**.
   - Under **Custom Parameter Locations in Response**, click **Add** to add our custom parameter, which will be used to extract the exact token we need.
   - At the bottom, you will notice the server response. Highlight the area that you want the macro to extract, and Burp Suite will auto-generate a regex pattern for you.
   - **Copy the regex pattern**, as we will need it later for the "Add Custom Header" extension.
   - Click **OK** to exit out of **Configure Item**, then click **OK** again to save and exit the **Macro Editor**, and finally click **OK** to save the macro.

   Great, we have created a macro that will extract the new JWT tokens. Next, we need to create a session handling rule.

3. **Create a Session Handling Rule**

   - While still in the Burp options **Sessions** window, scroll to the top and add a new session handling rule.
   - Give it a description and click **Add**; this will bring up a dropdown menu. Select **Check Session Is Valid**. This rule will first check if a session is valid by observing the HTTP response. If it gets a 401 HTTP response or if there is a unique way to identify that the session is invalid, we can have it run a macro to obtain the new JWT token.
   - In the next window, we can leave most settings as default. The important things to change here are the following:
     - Under **Inspect Response to Determine Session Validity**:
       - **Where to check**: Select **HTTP headers**. In my case, I get the "HTTP/1.1 401 Unauthorized" when a token is expired.
       - **Match Type**: **Literal String**.
       - **Case-Sensitivity**: **Insensitive**.
       - **Match Indicates**: **Invalid Session**.
   - Check the "If session is invalid, perform the action below" option and select **Run a macro**. Choose the macro you created, which will extract the token.
   - Ignore **Update current request with parameters** and **Update current request with cookies**; if they are selected, just ignore them.
   - At the bottom, check the box "After running the macro, invoke a Burp extension action handler:" and select **Add Custom Header**, the extension we installed.

   - We should now have our rule action added. At the top of the window, select **Scope**.
     - Use **Suite scope** (defined by the target tab). But if you're testing this, you can leave it to include all URLs.
     - Select **Tools scope** (Target, Intruder, Scanner), whichever you need.
   - Click **OK** to save and exit the session handling rule settings.

4. **Add the Regex Pattern to the Burp Extension "Add Custom Header"**

   - Navigate to the **Add Custom Header** extension tab.
   - Under **Header Value**, select **Regular Expression** and include the same regular expression you obtained from the macro.
   - The remaining settings can be left as default; they should be appropriate if your Authorization header uses Bearer. Click **Update Preview**.
   - Finally, we can confirm the setup by sending an expired token request. In the **Repeater** tab, make an API call and see if it updates the token.

**Note:** If you run into issues, check out the extension **Flow**, which can show you the exact HTTP requests being sent. This can help you tweak your session handling rules or macro.

### Conclusion

By setting up a macro and session handling rules in Burp Suite, you can automate the process of refreshing JWT tokens. This allows you to focus on testing the application without manual interruptions every time the token expires. Happy hacking!
