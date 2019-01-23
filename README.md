# prudhvibot


		========================================START BUILDING A SIMPLE CHATBOT===========================================

1. Install Ngrok.
2. Expose a port [eg:8080/8090....]

	$ command: ngrok http 8080

3. When we have exposed some port then we will able to access that port from the URL similar to below one.

		https://47ba4dd4.ngrok.io

4. Install Node.js [Latest Version if possible].
5. Create our app directory and set up our 'Chatbot' app.

		$ mkdir prudhvibot/

6. Initilize 'npm' inside the Directory.

		$ npm init

7. Once we configure our app, install 'Express' and 'body-parser'.

		$ npm install express body-parser --save

8. Create a 'webhook.js', and instantiate express and listen the server to port 8080, or whatever the port we have set with 'ngrok'.

	const express = require('express');
	const bodyParser = require('body-parser');
	const app = express();
	app.use(bodyParser.json());
	app.use(bodyParser.urlencoded({ extended: true }));

	const server = app.listen(process.env.PORT || 8080, () => {
  	console.log('Express server listening on port %d in %s mode', server.address().port, app.settings.env);
	});

9. create HTTP GET and POST route method to handle the command.

	/* For Facebook Validation */
	app.get('/webhook', (req, res) => {
  	if (req.query['hub.mode'] && req.query['hub.verify_token'] === 'tuxedo_cat') {
    	res.status(200).send(req.query['hub.challenge']);
  	} else {
    	res.status(403).end();
  		}
	});

	/* Handling all messenges */
	app.post('/webhook', (req, res) => {
  	console.log(req.body);
  	if (req.body.object === 'page') {
    	req.body.entry.forEach((entry) => {
      	entry.messaging.forEach((event) => {
        	if (event.message && event.message.text) {
          	sendMessage(event);
        	}
      	     });
    	   });
    	res.status(200).end();
  	  }
	});

10. This is how we receive messages to our webhook via Facebook Messenger- All requests should come via post, while the GET route is only used at the time we configure our Facebook app.

	Where we see the tuxedo_cat, just use an arbitrary string. we will need later when setting up our Facebook app.

11. Run the code, and go to the next step.

		$ node webhook.js

				=============================SETTING FACEBOOK PAGE APP================================

12. we need a Facebook Page to set up our chat bot. Create one from

		https://www.facebook.com/pages/create

 	Choose a category, and select a sub category from the dropdown and fill out the required filed. Then click Get Started.

13. Then create an app at developers.
	
		https://www.facebook.com/quickstarts

14. Once our app is created, follow the steps to configure or skip it to our Dashboard.

15. Go to Add Product from the left menu, then choose Messenger. Click Get Started.

16. At the Token Generation, (1) choose the page we just created from the dropdown menu, and it will generate a token (2) that we will need 	  to include in our node code.
    Then, at the Webhooks, (3) click the Setup Webhooks button:

17. In the dialog, fill out the (1) Callback URL with our ngrok URL, (2) the random string for validation (the one we’ve specified in our 	 ‘GET’ route in the webhook.js), then (3) check messages.

18. Click the Verify and Save. If we get a red icon with x at the Callback URL, it means the URL is not validated- either the URL is wrong  	 or our node code is not properly running. Otherwise, we are ready to get back to code.

19. Writing a Super Simple Chat Bot.

		$ npm install request --save

20. Continue with our webhook.js, let’s implement sendMessage() that simply replies the sender an echo to just test our Messenger bot.

	const request = require('request');

	function sendMessage(event) {
  	let sender = event.sender.id;
  	let text = event.message.text;

 	 request({
	    url: 'https://graph.facebook.com/v2.6/me/messages',
	    qs: {access_token: PAGE_ACCESS_TOKEN},
	    method: 'POST',
	    json: {
	      recipient: {id: sender},
	      message: {text: text}
	    }
	  }, function (error, response) {
	    if (error) {
	        console.log('Error sending message: ', error);
	    } else if (response.body.error) {
	        console.log('Error: ', response.body.error);
	    }
	  });
	}

21. Where we see the PAGE_ACCESS_TOKEN, use the generated token.Try running the code. This acts as a very simple bot, which interact the    	 Messenger platform to receive a message and echo the message as the reply- Go to https://m.me/our-PAGE-HASH and start a conversation. If 	  our the simple bot works correctly, it just replies the exactly what we send.

22. Now, our entry will be simply echoed back. And let’s use API.ai next to make this conversation more interesting.

23. Using API.ai with our Facebook Messenger Bot:
	API.ai allows developers to integrate our app with the AI system with speech-to-text and natural language processing.
	Let’s get started with API.ai by signing up.
	Once we get our account, create an agent. we can either click the button that says CREATE AGENT or from the menu.

24. Give it a name and fill out the required info.
    Make sure to click the Save button on the top every time we make changes.

			================================Making “Small Talk” with our Messenger Bot==========================

25. Instead of having our bot just echo we, let’s give it the API.ai’s Small Talk feature. This gives our bot an ability to have simple    	   conversations. From the left menu (if the menu is not visible, click the “Settings menu icon” at the top left to open), click Domains,  	  then activate the Small Talk.

26. Once activated, click View details then turn on the Fulfillment so that we can later use this feature in our app and customize.

			===============================use this feature in our bot==================================
Using API.ai Node Library.

27. It is actually possible to integrate API.ai with FB Messenger without programming, however, to make it fully customize in the way we want 	   to interact with our FB app, let’s use the service programmatically.
    First, install API.ai node.js library:
		
		$ npm install apiai

28. we need our API.ai API key and API secret to use the service with our bot. From the menu, click the “Config” icon to get our API key  	  (“Client access token”):

		Client Access Token : 713606e84c0544d0a73a0e14fff98ea2
29. And go back to our webhook.js and initialize apiai with the API key.

		const apiaiApp = require('apiai')(CLIENT_ACCESS_TOKEN);

30. we just passed a text to API.ai, and do something when we get the response (the response event).

31. Now, let’s modify the sendMessage().

	function sendMessage(event) {
	  let sender = event.sender.id;
	  let text = event.message.text;

	  let apiai = apiaiApp.textRequest(text, {
	    sessionId: 'tabby_cat' // use any arbitrary id
	  });

	  apiai.on('response', (response) => {
	    // Got a response from api.ai. Let's POST to Facebook Messenger
	  });

	  apiai.on('error', (error) => {
	    console.log(error);
	  });

	  apiai.end();
	}

32. What the code sample above doing is basically getting the info (passed as a param) sent from a user via Messenger, and pass the text     	 content to API.ai. Once the API.ai returns the answer, the response event is triggered.

33. Where we see the
		// Got a response…
    comment in the code sample above, let’s POST the response to the Messenger API.

	...
	apiai.on('response', (response) => {
	  let aiText = response.result.fulfillment.speech;

	    request({
	      url: 'https://graph.facebook.com/v2.6/me/messages',
	      qs: {access_token: PAGE_ACCESS_TOKEN},
	      method: 'POST',
	      json: {
	        recipient: {id: sender},
	        message: {text: aiText}
	      }
	    }, (error, response) => {
	      if (error) {
	          console.log('Error sending message: ', error);
	      } else if (response.body.error) {
	          console.log('Error: ', response.body.error);
	      }
	    });
	 });

34. Now, let’s test our bot. Run the node code, and try sending some messages. If everything is working, we should get replies from the bot.

		=====================================================END===================================================
