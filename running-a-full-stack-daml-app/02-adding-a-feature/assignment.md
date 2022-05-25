---
slug: adding-a-feature
id: 4gt8xuj2b4ry
type: challenge
title: Add a new feature with UI elements to your Daml app
notes:
- type: text
  contents: |
    This is the second to last in a series of four tutorials to get you up and running with full-stack Daml development. We do this through the example of a simple social networking application. The four tutorials cover:

    1. How to build and run the application ([Running the app](https://digitalasset.com/developers/interactive-tutorials/getting-started/build-and-run/)).
    1. The design of its different components ([Application Architecture](https://docs.daml.com/getting-started/app-architecture.html)).
    1. How to write a new feature for the app (this tutorial).
    1. How to deploy your app to [Daml Hub](https://www.digitalasset.com/developers/interactive-tutorials/getting-started/deploy-to-dabl/)

    In this scenario we'll dive into implementing a new feature for our social network app. This will give us a better idea how to develop Daml applications using our template.

    At the moment, our app lets us follow users in the network, but we have no way to communicate with them! Let’s fix that by adding a direct messaging feature. This will allow users that follow each other to send messages, respecting authorization and privacy. This means:

    - You cannot send a message to someone unless they have given you the authority by following you back.
    - You cannot see a message unless you sent it or it was sent to you.
    - We will see that Daml lets us implement these guarantees in a direct and intuitive way.

    There are three parts to building and running the messaging feature:

    1. Adding the necessary changes to the Daml model.
    2. Making the corresponding changes in the UI.
    3. Running the app with the new feature.

    As usual, we must start with the Daml model and base our UI changes on top of that.
tabs:
- title: Sandox
  type: terminal
  hostname: vsc-ide
- title: Terminal - UI
  type: terminal
  hostname: vsc-ide
- title: VSC IDE
  type: service
  hostname: vsc-ide
  port: 8080
- title: Daml UI
  type: service
  hostname: vsc-ide
  path: /damlui
  port: 3000
difficulty: basic
timelimit: 120000
---
Daml Changes
============

The Daml code defines the data and workflow of the application; you can read about this in more detail in the [application architecture](https://docs.daml.com/getting-started/app-architecture.html). The workflow aspect refers to the interactions between parties that are permitted by the system. In the context of a messaging feature, these are essentially the authorization and privacy concerns listed above.

For the authorization part, we take the following approach: a user Bob can message another user Alice when Alice starts following Bob back. When Alice starts following Bob back, she gives permission and authority to Bob to send her a message.

To implement this workflow, let’s start by adding the new data for messages.

- First, **open the IDE tab and wait for it to load.** You should see the IDE, similiar to the screen below, before proceeding with the next step.

- After that `copy` the below code to add the snippet to the end of the `User.daml` file. Indentation is important: it should be at the top level like the original User template.

```
template Message with
    sender: Party
    receiver: Party
    content: Text
  where
    signatory sender, receiver
```

This template is very simple: it contains the data for a message and no choices. The interesting part is the signatory clause: both the sender and receiver are signatories on the template. This enforces the fact that creation and archival of `Message` contracts must be authorized by both parties.

Now we can add messaging into the workflow by adding a new choice to the `User` template.

- Open the `daml/User.daml` file in the IDE
- Copy the below choice

```
    nonconsuming choice SendMessage: ContractId Message with
          sender: Party
          content: Text
        controller sender
        do
          assertMsg "Designated user must follow you back to send a message" (elem sender following)
          create Message with sender, receiver = username, content
```
- Finally paste the code into the `User` template after the `Follow` choice. **The indentation for the SendMessage choice must match the one of Follow. Make sure you save the file after copying the code.**

As with the `Follow` choice, there are a few aspects to note here.

- By convention, the choice returns the `ContractId` of the resulting `Message` contract.
- The parameters to the choice are the sender and content of this message; the receiver is the party named on this `User` contract.
- The controller clause states that it is the sender who can exercise the choice.
- The body of the choice first ensures that the sender is a user that the receiver is following and then creates the `Message` contract with the receiver being the signatory of the `User` contract.
- This completes the workflow for messaging in our app. Now let’s integrate this functionality into the UI.

Starting the Sandbox
====================

Next let's run the `daml start` command from the project's root folder.

```
cd create-daml-app
daml start
```

As a reminder the commmand will:

- Compile our Daml code into a DAR file containing the new feature
- Update the JavaScript library under ui/daml.js to connect the UI with your Daml code
- Upload the new DAR file to the Daml Sandbox

As mentioned previously, Daml Sandbox uses an in-memory store, which means it loses its state – which here includes all user data and follower relationships – when stopped or restarted.

Now let’s integrate the new functionality into the UI.

UI Changes
==========

The UI for messaging will consist of a new Messages panel in addition to the Follow and Network panel. This panel will have two parts:

- A list of messages you’ve received with their senders.
- A form with a dropdown menu for receiver selection and a text field for composing the message.

We will implement each part as a React component, which we’ll name `MessageList` and `MessageEdit` respectively. Let’s start with the simpler `MessageList`.

MessageList Component
=====================

The `MessageList` component queries all `Message` contracts where the receiver is the current user, and display their contents and senders in a list. The entire component is shown below.

First, we have created a new empty file `MessageList.tsx` in the `ui/src/components` folder.

Copy the below code into the `MessageList.tsx` file by clicking on the `Copy to Editor` button. Make sure to save the file in order for the changes to take effect.

```
import React from 'react'
import { List, ListItem } from 'semantic-ui-react';
import { User } from '@daml.js/create-daml-app';
import { userContext } from './App';

type Props = {
  partyToAlias: Map<string, string>
}
/**
 * React component displaying the list of messages for the current user.
 */
const MessageList: React.FC<Props> = ({partyToAlias}) => {
  const messagesResult = userContext.useStreamQueries(User.Message);

  return (
    <List relaxed>
      {messagesResult.contracts.map(message => {
        const {sender, receiver, content} = message.payload;
        return (
          <ListItem
            className='test-select-message-item'
            key={message.contractId}>
            <strong>{partyToAlias.get(sender) ?? sender} &rarr; {partyToAlias.get(receiver) ?? receiver}:</strong> {content}
          </ListItem>
        );
      })}
    </List>
  );
};

export default MessageList;
```

This is how the code works: The `messagesResult` gets the stream of all `Message` contracts where the receiver is our username. The streaming aspect means that we don’t need to reload the page when new messages come in. We extract the payload of every `Message` contract (the data as opposed to metadata like the contract ID) in messages. The rest of the component simply constructs a React List element with an item for each message.

There is one important point about privacy here. No matter how we write our `Message` query in the UI code, it is impossible to break the privacy rules given by the Daml model. That is, it is impossible to see a `Message` contract of which you are not the sender or the receiver (the only parties that can observe the contract). This is a major benefit of writing apps on Daml: the burden of ensuring privacy and authorization is confined to the Daml model.

MessageEdit Component
=====================

Next we need the `MessageEdit` component to compose and send messages to our followers. Again we show the entire component here.

First, we have created a new empty file `MessageEdit.tsx` in the `ui/src/components` folder. Copy the below code into the `MessageEdit.tsx` file by clicking on the `Copy to Editor` button. Make sure to save the file in order for the changes to take effect.

```
import React from 'react'
import { Form, Button } from 'semantic-ui-react';
import { Party } from '@daml/types';
import { User } from '@daml.js/create-daml-app';
import { userContext } from './App';

type Props = {
  followers: Party[];
  partyToAlias: Map<string, string>;
}

/**
 * React component to edit a message to send to a follower.
 */
const MessageEdit: React.FC<Props> = ({followers, partyToAlias}) => {
  const sender = userContext.useParty();
  const [receiver, setReceiver] = React.useState<string | undefined>();
  const [content, setContent] = React.useState("");
  const [isSubmitting, setIsSubmitting] = React.useState(false);
  const ledger = userContext.useLedger();

  const submitMessage = async (event: React.FormEvent) => {
    try {
      event.preventDefault();
      if (receiver === undefined) {
        return;
      }
      setIsSubmitting(true);
      await ledger.exerciseByKey(User.User.SendMessage, receiver, {sender, content});
      setContent("");
    } catch (error) {
      alert(`Error sending message:\n${JSON.stringify(error)}`);
    } finally {
      setIsSubmitting(false);
    }
  };

  return (
    <Form onSubmit={submitMessage}>
      <Form.Select
        fluid
        search
        className='test-select-message-receiver'
        placeholder={receiver ? partyToAlias.get(receiver) ?? receiver : "Select a follower"}
        value={receiver}
        options={followers.map(follower => ({ key: follower, text: partyToAlias.get(follower) ?? follower, value: follower }))}
        onChange={(event, data) => setReceiver(data.value?.toString())}
      />
      <Form.Input
        className='test-select-message-content'
        placeholder="Write a message"
        value={content}
        onChange={event => setContent(event.currentTarget.value)}
      />
      <Button
        fluid
        className='test-select-message-send-button'
        type="submit"
        disabled={isSubmitting || receiver === undefined || content === ""}
        loading={isSubmitting}
        content="Send"
      />
    </Form>
  );
};

export default MessageEdit;
```

You will first notice a `Props` type near the top of the file with a single following field. A prop in React is an input to a component; in this case a list of users from which to select the message receiver. The prop will be passed down from the `MainView` component, reusing the work required to query users from the ledger. You can see this following field bound at the start of the `MessageEdit` component.

We use the React `useState` hook to get and set the current choices of message receiver and content. The Daml-specific `useLedger` hook gives us an object we can use to perform ledger operations. The call to `ledger.exerciseByKey` in `sendMessage` looks up the `User` contract with the receiver’s username and exercises `SendMessage` with the appropriate arguments. The `sendMessage` wrapper reports potential errors to the user, and `submitMessage` additionally uses the `isSubmitting` state to ensure message requests are processed one at a time. The result of a successful call to `submitMessage` is a new `Message` contract created on the ledger.

The return value of this component is the React `Form` element. This contains a dropdown menu to select a receiver from the following, a text field for the message content, and a Send button which triggers `submitMessage`.

There is again an important point here, in this case about how authorization is enforced. Due to the logic of the `SendMessage` choice, it is impossible to send a message to a user who is not following us (even if you could somehow access their `User` contract). The assertion that elem sender following in `SendMessage` ensures this: no mistake or malice by the UI programmer could breach this.

MainView Component
==================

Finally we can see these components come together in the `MainView` component. We want to add a new panel to house our messaging UI. Open the `ui/src/components/MainView.tsx` file and start by adding imports for the two new components.

```
import MessageEdit from './MessageEdit';
import MessageList from './MessageList';
```

Next, find where the `Network Segment` closes, towards the end of the component. This is where we’ll add a new `Segment` for `Messages`. Make sure you’ve saved the file after copying the code.

```
            <Segment>
              <Header as='h2'>
                <Icon name='pencil square' />
                <Header.Content>
                  Messages
                  <Header.Subheader>Send a message to a follower</Header.Subheader>
                </Header.Content>
              </Header>
              <MessageEdit
                followers={followers.map(follower => follower.username)}
                partyToAlias={partyToAlias}
              />
              <Divider />
              <MessageList partyToAlias={partyToAlias}/>
            </Segment>
```

You can see we simply follow the formatting of the previous panels and include the new messaging components: `MessageEdit` supplied with the usernames of all visible parties as props, and `MessageList` to display all messages.

That is all for the implementation! Let’s give the new functionality a spin.

Running the new feature
=======================

It's time to run the app with the newly added feature!

As we're running the build process from scratch for the project, we need to install the project dependencies again. This wouldn't be necessary in case where project dependencies would have been previoulsy installed.

Let's install the dependencies by running the below commands in the second terminal:

```
cd create-daml-app/ui
npm install
```

Now we're ready to start the UI by running:

```
npm start
```

As a reminder, this starts the web UI connected to the running Daml Sandbox and JSON API server. In case where we would have an already running UI this step wouldn't be necessary.

Once the web UI has been compiled and started, you should see `Compiled successfully!` in your terminal. You can now [open the UI tab](https://[HOSTNAME]-[PORT]-[PARTICIPANT_ID].env.play.instruqt.com), where you should see the same login page as before

Once you’ve logged in, you’ll see a familiar UI but with our new Messages panel at the bottom!

Go ahead and follow some more users. Then, log in as some of those users by [right clikcing on this link and opening a separate tab in your browser](https://[HOSTNAME]-[PORT]-[PARTICIPANT_ID].env.play.instruqt.com) to follow yourself back. Then, if you click on the dropdown menu in the Messages panel, you’ll be able to see some followers to message!


Send some messages between users and make sure you can see each one from the other side. You’ll notice that new messages appear in the UI as soon as they are sent (due to the streaming React hooks).

