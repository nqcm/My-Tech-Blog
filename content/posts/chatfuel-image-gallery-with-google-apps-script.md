---
title: "🔥 Chatfuel Image Gallery from Google Sheets with Google Apps Script"
date: 2019-07-04T13:25:09+02:00
tags:
  [
    "Bots",
    "Messenger",
    "Chatfuel",
    "Google Sheets",
    "Google Apps Script",
    "JavaScript",
  ]
summary: A step by step guide to building an image gallery from Google Sheets without using third party integrations. Increasing the functionality of your Chatfuel bot using Google Sheets as a Database for a Chatfuel Bot.
cover:
  image: "/img/googleAppScriptWithBot/gallery-from-sheets.png"
---

<br>

![Image gallery from Google Sheets](/img/googleAppScriptWithBot/chatfuel-app-with-google-apps-script.png#center)

Let’s say you are making a bot for a restaurant on [Chatfuel](https://chatfuel.com). The bot can tell the users the daily specials. The daily specials would change, well, daily and you want the restaurant staff to update them. Changing things daily inside Chatfuel by people who may not be tech savvy, the restaurant staff for example, is not the recommended thing to do. And things can get further complicated if you are making the bot for a client.

So, we want a place where the staff can update the specials every day and the bot reads the specials before sending the information to the user. Typically, this is a job for a database, but Google Sheets can be used as a light weight and easy to use alternative to databases in this scenario.

The only problem is that Chatfuel doesn’t offer the functionality to read data from Google Sheets out of box. Using third party integrations, [Zapier](https://zapier.com/) or [Integromat](https://www.integromat.com/en/) for instance, does the trick but it adds to the overhead costs.

The good news is that if you know some basic JavaScript, you can use Google Sheets like a regular database and integrate it with your bot. Enter [Google Apps Script](https://developers.google.com/apps-script/)!

> Google Apps Script lets you do new and cool things with Google Sheets. You can use Apps Script to add custom menus, dialogs, and sidebars to Google Sheets. It also lets you write custom functions for Sheets, as well as integrate Sheets with other Google services like Calendar, Drive, and Gmail.

But Google Apps Script can do a lot more than that. And in this tutorial, we will use Google Apps Script to read the daily specials for a pizza shop from a Google Sheet and send the data as gallery cards to Messenger through our bot.

If you are not familiar with Google Apps Script, it is a scripting language for light-weight application development in the Google ecosystem. It is based on JavaScript. So, if you are familiar with JavaScript, using Google Apps Script is fairly simple.

Let’s get started.

## Scaffolding

Go to Google Sheets and create a new blank sheet.

![Create a blank Google sheet](/img/googleAppScriptWithBot/create-sheet.png#center)

To follow along this tutorial, make columns for name, description and image URL. Here is a screen shot of my sheet with some fake data. Make sure your images are hosted somewhere on the web and they have the right permissions.

![Sample sheet](/img/googleAppScriptWithBot/populate-sheet.png#center)

Once your sheet is set up as you want, let’s write our script.

## Introducing Google Apps Script

There are different types of Google Apps scripts and in this tutorial, I will create a [container bound script](https://developers.google.com/apps-script/guides/bound). You can read more about the different kinds of scripts [here](https://developers.google.com/apps-script/guides/standalone). But basically, what it means is that a script which is bound to a Google Sheet cannot be detached from the file they are bound to, and they gain a few special privileges over the parent file.

To create a bound script, in your Google Sheet, select tools from the menu and then select Script Editor.

![Open script editor from tools](/img/googleAppScriptWithBot/select-script-editor.png#center)

It will open the Google Apps Scripts project page.

> A project represents a collection of files and resources in Google Apps Script, sometimes referred to simply as "a script". A script project has one or more script files which can either be code files (having a .gs extension) or HTML files (a .html extension). You can also include JavaScript and CSS in HTML files.

You can read more about Google Apps Scripts projects [here](https://developers.google.com/apps-script/guides/projects).

Give your project a suitable name.

![Project page screen shot](/img/googleAppScriptWithBot/code-editor-screenshot.png#center)

As you can see there is a code editor where we will write our code. Currently there is just an empty function here.

```javascript
function myFunction() {}
```

Google Apps Script has a basic logging mechanism using the `Logger` class. So we can use `Logger.log` in place of JavaScript’s `console.log`. Let’s log a simple “Hello, world!”.

```javascript
function myFunction() {
  Logger.log("Hello World!");
}
```

Click save and then run your script.

![Save your project and run the function](/img/googleAppScriptWithBot/save-run.png#center)

Now click on View > Logs or simply hit Ctrl + Enter and you should see Hello World! displayed on the logs.

![View logs](/img/googleAppScriptWithBot/view-logs.png#center)

![Logs](/img/googleAppScriptWithBot/hello-world-log.png#center)

## Deploying The Script As A Web App

At the moment this is just a script. We will need to turn this script into a [web app](https://developers.google.com/apps-script/guides/web) so our bot can communicate with it.

Any script can be published as a web app if it meets these requirements:

- It contains a doGet(e) or doPost(e) function.
- And the function returns an HTML service HtmlOutput object or a Content service TextOutput object.

Armed with this information, lets change our function.

```javascript
function doGet() {
  Logger.log("Hello World!");
  return ContentService.createTextOutput("Hello, world!");
}
```

Let’s save this script again and then deploy it as a web app. Hit Publish and select "Deploy as web app".

![Deploy as web app](/img/googleAppScriptWithBot/deploy-as-web-app.png#center)

Change the “Who has access to this app” to “Anyone, even anonymous” and click deploy.

![Deploying](/img/googleAppScriptWithBot/deploy1.png#center)

Note the web app URL from the next screen.

## Integrating with Chatfuel

Now go to your Chatfuel bot and add a JSON API card where you want to display the specials. Change the Type of the request to GET and in the URL paste the URL you copied from the Google Apps Script project page. Test the request. In the response section, under Response Body, you should see our Hello, world! text.

![Chatfuel Json API card](/img/googleAppScriptWithBot/test-json-connection.png#center)

Now that our bot is connected to our web app running on Google Apps Script project, let’s see how we can read data from the Google Sheets.

## Reading Data From Google Sheets Programmatically

To retrieve the data from the spreadsheet, you must get access to the spreadsheet where the data is stored, get the range in the spreadsheet that holds the data, and then get the values of the cells.

> Apps Script facilitates access to the data by reading structured data in the spreadsheet and creating JavaScript objects for them.

Since we are using a bound script the above process is fairly easy. We will just call a few methods on the JavaScript object created for us. You can read about all available methods [here](https://developers.google.com/apps-script/reference/spreadsheet/spreadsheet-app). Change the code to following.

```javascript
function doGet() {
  var sheet = SpreadsheetApp.getActiveSheet();
  var data = sheet.getDataRange().getValues();
  for (var i = 0; i < data.length; i++) {
    Logger.log("Item name: " + data[i][0]);
    Logger.log("Item description: " + data[i][1]);
  }
  return ContentService.createTextOutput("Hello, world!");
}
```

Hit save. Google will ask you permission to access data and it may tell you that this web app is not safe. Proceed anyway and then run your function. Check the logs and you should get something like this.

![Logs](/img/googleAppScriptWithBot/logs1.png#center)

As you can see it is also reading the header row with the data. But that can be fixed easily by initializing our loop variable with 1 instead of 0.

This is an extremely simple script and we are just scratching the surface of all the possibilities offered to us. Feel free to play around with the code and build more complex functionality. But for the purpose of this tutorial, this script will do.

## Making Image Gallery From The Data

Now that we know how we can read and parse data from our sheet programmatically, let's see how we can send this data back as a gallery.

Chatfuel documentation gives us all the information we need. Go to the [JSON API section](https://docs.chatfuel.com/en/articles/735122-json-api) and scroll down to "Sending galleries". The page looks like this.

![Chatfuel docs](/img/googleAppScriptWithBot/chatfuel-docs.png#center)

As we can see, we need to send the actual data in the form of a list of objects.

So let's first create a list of objects or elements from our data.

Change the code to following.

```javascript
function doGet() {
  var sheet = SpreadsheetApp.getActiveSheet();
  var data = sheet.getDataRange().getValues();
  elements = create_elements(data);
  Logger.log(elements);
  return ContentService.createTextOutput("Hello, world!");
}

function create_elements(data) {
  var elements = [];
  for (var i = 1; i < data.length; i++) {
    var object = {
      title: data[i][0],
      image_url: data[i][2],
      subtitle: data[i][1],
      buttons: [
        {
          type: "web_url",
          url: "https://blog.naveeraashraf.com/",
          title: "View Item",
        },
      ],
    };
    elements.push(object);
  }
  return elements;
}
```

We are looping through our rows and adding the data to a JavaScript object, which is then pushed to a list. We also moved the code to create our objects in a separate function to keep our code clean. You can check your logs to see if your code is working properly. Make sure to change the url in the above code.

So far we are only logging the objects and not sending them to our bot. Let's change that. First we will use our objects to create the response which will build a gallery. Add the following function to your code. You can copy the response from [Chatfuel docs](https://docs.chatfuel.com/en/articles/735122-json-api) if you wish and make necessary changes.

```javascript
function buildImageGallery(elements) {
  var output = JSON.stringify({
    messages: [
      {
        attachment: {
          type: "template",
          payload: {
            template_type: "generic",
            image_aspect_ratio: "square",
            elements: elements,
          },
        },
      },
    ],
  });

  return ContentService.createTextOutput(output).setMimeType(
    ContentService.MimeType.JSON
  );
}
```

We are simply replacing the elements list in the docs with the list we have created in the previous step.

We will also add some functionality to our code for when there is no data in the sheet. This way our code won't break in case the restaurant staff forgot to add the new specials but deleted the old ones.

Your final code should look like this now.

```javascript
function doGet() {
  var sheet = SpreadsheetApp.getActiveSheet();
  var data = sheet.getDataRange().getValues();
  elements = create_elements(data);
  if (elements.length != 0) {
    return buildImageGallery(elements);
  } else {
    return notFound();
  }
}

function create_elements(data) {
  var elements = [];
  for (var i = 1; i < data.length; i++) {
    var object = {
      title: data[i][0],
      image_url: data[i][2],
      subtitle: data[i][1],
      buttons: [
        {
          type: "web_url",
          url: "https://blog.naveeraashraf.com/",
          title: "View Item",
        },
      ],
    };
    elements.push(object);
  }
  return elements;
}

function buildImageGallery(elements) {
  var output = JSON.stringify({
    messages: [
      {
        attachment: {
          type: "template",
          payload: {
            template_type: "generic",
            image_aspect_ratio: "square",
            elements: elements,
          },
        },
      },
    ],
  });

  return ContentService.createTextOutput(output).setMimeType(
    ContentService.MimeType.JSON
  );
}

function notFound() {
  var output = JSON.stringify({
    messages: [
      {
        text: "There are no items in this category",
      },
    ],
  });
  return ContentService.createTextOutput(output).setMimeType(
    ContentService.MimeType.JSON
  );
}
```

Test your bot in Messenger and you should get the data from your sheet displayed as an image gallery.

![Image gallery from Google Sheets](/img/googleAppScriptWithBot/pizza-bot-demo-1.png#center)

That's it! I hope you enjoyed this tutorial. If you did don't forget to share it.

<br>
