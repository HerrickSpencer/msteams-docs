---
title: Channel and Group chat conversations with bots
description: Describes the end-to-end scenario of having a conversation with a bot in a channel in Microsoft Teams
keywords: teams scenarios channels conversation bot
ms.date: 05/20/2019
---
# Channel and Group chat conversations with a Microsoft Teams bot

Microsoft Teams allows users to bring bots into their channel or group chat conversations. By adding a bot to a team or chat, all users of the conversation can take advantage of the bot functionality right in the conversation. You can also access Teams-specific functionality within your bot like querying team information and @mentioning users.

Chat in channels and group chats differ from personal chat in that the user needs to @mention the bot. If a bot is used in multiple scopes (personal, groupchat or channel) you will need to detect what scope the bot messages came from, and process them accordingly.

## Designing a great bot for channels or groups

Bots added to a team become another team member, who can be @mentioned as part of the conversation. In fact, bots only receive messages when they are @mentioned, so other conversations on the channel are not sent to the bot.

> [!NOTE]
> For convenience when replying to bot messages in a channel, the bot name is prepended in the compose message box automatically.

A bot in a group or channel should provide information relevant and appropriate for all members. While your bot can certainly provide any information relevant to the experience, keep in mind conversations with it are visible to everyone. Therefore, a great bot in a group or channel should add value to all users, and certainly not inadvertently share information more appropriate in a one-to-one conversation.

Your bot might be entirely relevant in both scopes (personal and channel) as is, and no significant extra work is required to allow your bot to work in both. In Microsoft Teams there is no expectation that your bot function in all scopes, but you should ensure your bot provides user value in whichever scope(s) you choose to support. For more information on scopes, see [Apps in Microsoft Teams](~/concepts/apps/apps-overview.md).

Developing a bot that works in groups or channels uses much of the same functionality from personal conversations. Additional events and data in the payload provide Teams group and channel information. Those differences, as well as key differences in common functionality are described in the following sections.

### Creating messages

For more information on bots creating messages in channels see [Proactive messaging for bots](~/concepts/bots/bot-conversations/bots-conv-proactive.md), and specifically [Creating a channel conversation](~/concepts/bots/bot-conversations/bots-conv-proactive.md#creating-a-channel-conversation).

### Receiving messages

For a bot in a group or channel, in addition to the [regular message schema](https://docs.botframework.com/en-us/core-concepts/reference/#activity), your bot also receives the following properties:

* `channelData` See [Teams channel data](~/concepts/bots/bot-conversations/bots-conversations.md#teams-channel-data). In a groupChat, contains information specific to that chat.
* `conversation.id` The reply chain ID, consisting of channel ID plus the ID of the first message in the reply chain
* `conversation.isGroup` Set to `true` for bot messages in channels
* `conversation.conversationType` Either `groupChat` or `channel`
* `entities` Can contain one or more mentions (see [Mentions](#-mentions))

### Replying to messages

To reply to an existing message, call [`ReplyToActivity`](/bot-framework/dotnet/bot-builder-dotnet-connector#send-a-reply) in .NET or [`session.send`](/bot-framework/nodejs/bot-builder-nodejs-use-default-message-handler) in Node.js. The Bot Builder SDK handles all the details.

If you choose to use the REST API, you can also call the [`/conversations/{conversationId}/activities/{activityId}`](/bot-framework/rest-api/bot-framework-rest-connector-send-and-receive-messages#send-the-reply) endpoint.

In a channel, replying to a message shows as a reply to the initiating reply chain. The `conversationId` contains the channel and the top level message ID. Although the Bot Framework takes care of the details, you can cache that `conversationId` for future replies to that conversation thread as needed.

### Best practice: Welcome messages in Teams

When your bot is first added to the group or team, it is generally useful to send a welcome message introducing the bot to all users. The welcome message should provide a description of the bot’s functionality and user benefits. Ideally the message should also include commands for the user to interact with the app. To do this, ensure that your bot responds to the `conversationUpdate` message, with the `teamsAddMembers` eventType in the `channelData` object. Be sure that the `memberAdded` ID is the bot's App ID itself, because the same event is sent when a user is added to a team. See [Team member or bot addition](~/concepts/bots/bots-notifications.md#team-member-or-bot-addition) for more details.

You might also want to send a personal message to each member of the team when the bot is added. To do this, you could [fetch the team roster](~/concepts/bots/bots-context.md#fetching-the-team-roster) and send each user a [direct message](~/concepts/bots/bot-conversations/bots-conv-proactive.md).

We recommend that your bot *not* send a welcome message in the following situations:

* The team is large (obviously subjective, but for example larger than 100 members). Your bot may be seen as 'spammy' and the person who added it may get complaints unless you clearly communicate your bot's value proposition to everyone who sees the welcome message.
* Your bot is first mentioned in a group or channel (versus being first added to a team)
* A group or channel is renamed
* A team member is added to a group or channel

For more best practices, see our [design guidelines](~/resources/design/overview.md).

## @ Mentions

Because bots in a group or channel respond only when they are mentioned ("@_botname_") in a message, every message received by a bot in a group channel contains its own name, and you must ensure your message parsing handles that. In addition, bots can parse out other users mentioned and mention users as part of their messages.

### Retrieving mentions

Mentions are returned in the `entities` object in payload and contain both the unique ID of the user and, in most cases, the name of user mentioned. You can retrieve all mentions in the message by calling the `GetMentions` function in the Bot Builder SDK for .NET, which returns an array of `Mentioned` objects.

#### .NET example code: Check for and strip @bot mention

```csharp
Mention[] m = sourceMessage.GetMentions();
var messageText = sourceMessage.Text;

for (int i = 0;i < m.Length;i++)
{
    if (m[i].Mentioned.Id == sourceMessage.Recipient.Id)
    {
        //Bot is in the @mention list.
        //The below example will strip the bot name out of the message, so you can parse it as if it wasn't included. Note that the Text object will contain the full bot name, if applicable.
        if (m[i].Text != null)
            messageText = messageText.Replace(m[i].Text, "");
    }
}
```

> [!NOTE]
> You can also use the Teams extension function `GetTextWithoutMentions`, which strips out all mentions, including the bot.

#### Node.js example code: Check for and strip @bot mention

```javascript
var text = message.text;
if (message.entities) {
    message.entities
        .filter(entity => ((entity.type === "mention") && (entity.mentioned.id.toLowerCase() === botId)))
        .forEach(entity => {
            text = text.replace(entity.text, "");
        });
    text = text.trim();
}
```

You can also use the Teams extension function `getTextWithoutMentions`, which strips out all mentions, including the bot.

### Constructing mentions

Your bot can mention other users in messages posted into channels. To do this, your message must do the following:

* Include `<at>@username</at>` in the message text
* Include the `mention` object inside the entities collection

The [Teams extensions for the Bot Builder SDK](~/get-started/code.md#microsoft-teams-extensions-for-the-bot-builder-sdk) provide functionality to easily implement this.

#### .NET example

This example uses the [Microsoft.Bot.Connector.Teams](https://www.nuget.org/packages/Microsoft.Bot.Connector.Teams) NuGet package.

```csharp
// Create reply activity
Activity replyActivity = activity.CreateReply();

// Construct text of the form @sender Hello
replyActivity.Text = "Hello ";
replyActivity.AddMentionToText(activity.From, MentionTextLocation.AppendText);

// Send the reply activity
await client.Conversations.ReplyToActivityAsync(replyActivity);
```

#### Node.js example

This sample uses the [botbuilder-teams](https://www.npmjs.com/package/botbuilder-teams) npm package.

```javascript
// User to mention
var toMention: builder.IIdentity = {
    name: 'John Doe',
    id: userId
};

// Create a message and add mention to it
var msg = new teams.TeamsMessage(session).text(teams.TeamsMessage.getTenantId(session.message));
var mentionedMsg = msg.addMentionToText(toMention);

// Post the message
var generalMessage = mentionedMsg.routeReplyToGeneralChannel();
session.send(generalMessage);
```

#### Example: Outgoing message with user mentioned

```json
{
    "type": "message", 
    "text": "Hey <at>Pranav Smith</at> check out this message",
    "timestamp": "2017-10-29T00:51:05.9908157Z",
    "localTimestamp": "2017-10-28T17:51:05.9908157-07:00",
    "serviceUrl": "https://skype.botframework.com",
    "channelId": "msteams",
    "from": {
        "id": "28:9e52142b-5e5e-4d7b-bb3e- e82dcf620000",
        "name": "SchemaTestBot"
    },
    "conversation": {
        "id": "19:aebd0ad4d6ab42c8b9ed19c251c2fc37@thread.skype;messageid=1481567603816"
    },
    "recipient": {
        "id": "8:orgid:6aebbad0-e5a5-424a-834a-20fb051f3c1a",
        "name": "stlrgload100"
    },
    "attachments": [
        {
            "contentType": "image/png",
            "contentUrl": "https://upload.wikimedia.org/wikipedia/en/a/a6/Bender_Rodriguez.png",
            "name": "Bender_Rodriguez.png"
        }
    ],
    "entities": [
        {
            "type":"mention",
            "mentioned":{
                "id":"29:08q2j2o3jc09au90eucae",
                "name":"Pranav Smith"
            },
            "text": "<at>@Pranav Smith</at>"
        }
    ],
    "replyToId": "3UP4UTkzUk1zzeyW"
}
```

## Accessing groupChat or channel scope

Your bot can do more than send and receive messages in groups and teams. For instance, it can also fetch the list of members, including their profile information, as well as the list of channels. See [Get context for your Microsoft Teams bot](~/concepts/bots/bots-context.md) to learn more.