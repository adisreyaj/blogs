## Get Signup Notifications In Telegram Using Auth0 Actions.

Auth0 actions are so powerful that they can be used to do a lot of cool things. Here's how you can send notifications to Telegram whenever a new user signs up. 

I recently worked on a small project which is a small e-commerce application built using Angular and NestJs. Auth0 is used for authenticating the users. I had a very interesting thought of adding notifications when a new user signs up. The easiest way for me was to use Auth0 Actions.

## Auth0 Actions
Actions are one of the coolest features of Auth0. I love it and have used it for multiple scenarios. Actions are custom Node.js functions that get executed during certain points like User Login, Registration, etc.

Here's a definition from the docs:

> Actions are functional services that fire during specific events across multiple identity flows.

**Auth0 hooks** allowed us to add custom logic that gets triggered when certain events happen. Actions are a more advanced version of hooks that provides more extensibility.

Official docs: https://auth0.com/docs/customize/actions

### Action Triggers
The custom functions that we write are called by certain events. Here are the supported triggers:

![Action triggers](https://cdn.hashnode.com/res/hashnode/image/upload/v1642323501366/zq800xdZ8.png)

Here are more details of when exactly these actions are triggered:
https://auth0.com/docs/customize/actions/triggers

## Implementing the Action
For our use case, we need the notification to be sent when a new user signs up. So we could use the **Post User Registration** trigger for our action.

### 1. Create a custom action
The first step is to create a new custom Action. We do that by navigating to `Actions > Custom` and then clicking on the **Build Custom** button.

![Actions](https://cdn.hashnode.com/res/hashnode/image/upload/v1642324835222/9LKAt5nwl.png)

We get a modal asking to give the Action a name and also select a Trigger and the Environment.

You can fill the form up with the following details:
- *Name*: **New User Notifications**
- *Trigger*: **Post User Registration**
- *Runtime*: **Node 16 (Recommended)**

### 2. Setting up pre-requisites
Once the action is created, you can see the list of the Actions under **Custom** tab on the Actions Library page.

![Custom Actions Page](https://cdn.hashnode.com/res/hashnode/image/upload/v1642324644640/8-U_hk8WF.png)

There are a few things that we need to do before we can start writing the actual logic.

#### Creating a Telegram Bot
Telegram is a really powerful messaging platform that can do a lot of stuff. One of the best things about it is that we can create bots and also send messages using the  [Telegram Bot API](https://core.telegram.org/bots/api) . 

Put a message to  [BotFather](https://t.me/botfather) on Telegram
```
/newbot
```

It will prompt to you give a name. Give a name and then you'll be given the **Bot Token**.
![telegram bot.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1642327890346/DXu0QEak1.png)

Now that we have the bot token, we can make API calls using the Telegram Bot API.

Ref: https://core.telegram.org/bots#3-how-do-i-create-a-bot

Before we can send a message to the Bot, we need to get the **Channel Id**. For that send a message to the bot and then just paste the following URL in the browser (replace the <bot-token> with yours):
```
https://api.telegram.org/bot<bot-token>/getUpdates
```
You will be able to see the message details that was sent to the bot:
```json
{
  "ok": true,
  "result": [
    {
      "update_id": 723563447,
      "message": {
        "message_id": 7,
        "from": {
          "id": 627445600, // <-- Copy this Id
          "is_bot": false,
          "first_name": "John Doe",
          "username": "johndoe",
          "language_code": "en"
        },
        "chat": {
          "id": 627445600,
          "first_name": "Jane Doe",
          "username": "janedoe",
          "type": "private"
        },
        "date": 1642266764,
        "text": "Test"
      }
    }
  ]
}
```
The `id` is the `channel_id` that we are going to use for sending messages.

### 3. Writing the Action Logic

Now that we have the things needed, let's start writing the logic. So here are the things that need to be set up in the actions.

#### Installing Dependencies
Actions allow us to install packages that we can use inside the function, in our case we need to make API requests to the Telegram Bot API to send messages. For that, we can install a library called `node-fetch`. 

To do so, go to your newly created action and click on the **Modules** section.

![Actions Modules](https://cdn.hashnode.com/res/hashnode/image/upload/v1642328919220/kcxIDQv9K.png)

**Note**: We install `node-fetch@2` explicitly because we want the `CommonJs` version of the library.

#### Adding Env Variables
Actions also have a way to keep our environment variables a secret. This is where we are going to save the **Bot Token** and the **Channel id** that we will use in the code. It's not a good idea to put them in the code as they are sensitive information.

There is a **Secrets** section under which we can save them. Create a secret for the token and the channel id.

![Actions Secrets](https://cdn.hashnode.com/res/hashnode/image/upload/v1642329296076/ZqDEWKjrK.png)

#### Writing the actual logic

Now you can use `node-fetch` to make a post request to the `/sendMessage` API endpoint.

```js
const request = require('node-fetch'); // <-- require the library

/**
* Handler that will be called during the execution of a PostUserRegistration flow.
*
* @param {Event} event - Details about the context and user that has registered.
*/
exports.onExecutePostUserRegistration = async (event) => {
  try{
    const {family_name, given_name} = event.user;
    await request(`https://api.telegram.org/bot${event.secrets.BOT_TOKEN}/sendMessage`, 
      {
        method:'POST',
        body: JSON.stringify({
          "chat_id": event.secrets.TELEGRAM_CHAT_ID,
          "text":`New User Signed up: ${given_name} ${family_name}`
        }),
        headers: {
          'content-type': 'application/json'
        }
      }
    );
  } catch(err){
    console.log('Failed to notify');
  }
};
```
Now the action can be deployed.

Ref: https://core.telegram.org/bots/api#sendmessage

### 4. Using the Action
Once the action is deployed, we can use it in a flow. To do so, navigate to the `Actions > Flows` Page and select **Post User Registration** flow from the cards.

![Auth0 flow](https://cdn.hashnode.com/res/hashnode/image/upload/v1642330322367/5kzrptgw-.png)

We can find the action that we built under that **Custom** tab. Dragging and dropping the action to the flow does the job of activating it. The only thing left now is to just **Apply** the flow.

We are done setting up.

So now whenever someone signs up, you get a message in Telegram.

![Notification](https://cdn.hashnode.com/res/hashnode/image/upload/v1642330497846/LSvxZsnUp.png)

There are tons of cool use-cases for Actions. If you would like to see more blogs on it, do let me know.

## Connect with me

- [Twitter](https://twitter.com/AdiSreyaj)
- [Github](https://github.com/adisreyaj)
- [Linkedin](https://www.linkedin.com/in/adithyasreyaj/)
- [Cardify](https://cardify.adi.so) - Dynamic SVG Images for Github Readmes


Do add your thoughts in the comments section.
Stay Safe ❤️

[![Buy me a pizza](https://cdn.hashnode.com/res/hashnode/image/upload/v1639498527478/IA3aJ9R0J.png)](https://www.buymeacoffee.com/adisreyaj)
