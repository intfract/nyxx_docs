---
title: Message components
author: HarryET
timestamp: 2021-09-22
category: guides
sidebar_position: 5
---

Message components allow for interactivity between a message sent by a bot and the user receiving it. You can add buttons, links, select menus & multiselects.

<br />

## Interactions extension

Before you use message components you have to instantiate new instance of the [`IInteractions`] class, which is an extension for nyxx that provides slash command and message components functionality:

```dart
final bot = NyxxFactory.createNyxxWebsocket("<TOKEN>", GatewayIntents.allUnprivileged);
final interactions = IInteractions.create(WebsocketInteractionBackend(bot));
```

<br />

The [`IInteractions`] class contains all the methods and utils needed to send and manage messages with components.

<br />

### Components (buttons and select menus)

Components (buttons and select menus at the moment) can be created on any message but button events can be listened only from the [`IInteractions`] object.

<br />

### Adding components to message

Components can be created using the [`ComponentMessageBuilder`] class from the [`nyxx_interactions`] package. It extends the basic
[`MessageBuilder`] from the [`nyxx`] package with an additional [`addComponent`] method which allows adding component rows and
components to a message.

The following example shows how to create a message with buttons and select menus. These can be sent with any message
sent by the bot, including webhooks (provided the bot application also owns the webhook):

<br />

```dart
final singleCommand = SlashCommandBuilder("help", "This is example help command", [])
  ..registerHandler((event) async {
    // All "magic" happens via ComponentMessageBuilder class that extends MessageBuilder
    // from main nyxx package. This new builder allows to create message with components.
    final componentMessageBuilder = ComponentMessageBuilder();

    // There are two types of button - regular ones that can be responded to an interaction
    // and url button that only redirects to specified url.
    // Here we are focusing on regular button that we can respond to.
    // Label is what user will see on button, customId is id that we ca use later to
    // catch button event and respond to, and style is what kind of button we want create.
    final componentRow = ComponentRowBuilder()
      ..addComponent(ButtonBuilder("This is button label", "thisisid", ButtonStyle.success))
      ..addComponent(ButtonBuilder("This is another button", "thisisid2", ButtonStyle.success))
      ..addComponent(ButtonBuilder("This is a disabled button", "thisisid3", ButtonStyle.danger)
        ..disabled = true,
    );

    // Then component row can be added to message builder and sent to user.
    // You MUST add content to the MessageBuilder class, otherwise, Discord will throw an error.
    componentMessageBuilder.addComponentRow(componentRow..content = 'Hey, I got buttons!');
    await event.respond(componentMessageBuilder);
  });
```

<br />
Of course, you can do this with a select menu.

```dart
final singleCommand = SlashCommandBuilder("help", "This is example help command", [])
  ..registerHandler((event) async {
    // All "magic" happens via ComponentMessageBuilder class that extends MessageBuilder
    // from main nyxx package. This new builder allows to create message with components.
    final componentMessageBuilder = ComponentMessageBuilder();

    // Adding selects is as easy as adding buttons. Use MultiselectBuilder with custom id
    // and list of multiselect options.
    final componentRow = ComponentRowBuilder()
      ..addComponent(MultiselectBuilder("customId", [
        MultiselectOptionBuilder("example option 1", "option1"),
        MultiselectOptionBuilder("example option 2", "option2"),
        MultiselectOptionBuilder("example option 3", "option3"),
      ]));
    // Then component row can be added to message builder and sent to user.
    // You MUST add content to the MessageBuilder class, otherwise, Discord will throw an error.
    componentMessageBuilder.addComponentRow(componentRow..content = 'Hey, I got a select menu!');
    await event.respond(componentMessageBuilder);
```

<br />
Or add a custom placeholder:

```dart
final componentRow = ComponentRowBuilder()
      ..addComponent(MultiselectBuilder("customId", [
        MultiselectOptionBuilder("example option 1", "option1"),
        MultiselectOptionBuilder("example option 2", "option2"),
        MultiselectOptionBuilder("example option 3", "option3"),
      ])
      // Use the setter `placeholder` to set a custom placeholder
      //highlight-next-line
      ..placeholder = "A custom placeholder",
  );

componentMessageBuilder.addComponentRow(componentRow..content = 'Hey, I got a select menu!');
await event.respond(componentMessageBuilder);
```

<br />
You can even disable the select menu:

```dart
final componentRow = ComponentRowBuilder()
      ..addComponent(MultiselectBuilder("customId", [
        MultiselectOptionBuilder("example option 1", "option1"),
        MultiselectOptionBuilder("example option 2", "option2"),
        MultiselectOptionBuilder("example option 3", "option3"),
      ])
      // Use the setter `disabled` to disable the select menu
      //highlight-next-line
      ..disabled = true,
  );

componentMessageBuilder.addComponentRow(componentRow..content = 'Hey, I got a select menu!');
await event.respond(componentMessageBuilder);
```

<br />

### Listening to component events

If your custom id doesn't hold specific data, you can use [`IInteractions.registerButtonHandler`] or [`IInteractions.registerMultiselectHandler`]
to listen for specific events. If your custom id for the component holds, for example, state data you can listen to
[`IInteractions.events.onMultiselectEvent`] or [`IInteractions.events.onButtonEvent`] events and then handle the interaction yourself.

```dart
// To handle button interaction you need need function that accepts
// ButtonInteractionEvent as parameter. Since button event is interaction like
// slash command it needs to acknowledged and/or responded.
// If you know that command handler would take more that 3 second to complete
// you would need to acknowledge and then respond later with proper result.
Future<void> buttonHandler(ButtonInteractionEvent event) async {
  await event.acknowledge(); // ack the interaction so we can send response later

  // Send followup to button click with id of button
  await event.sendFollowup(MessageBuilder.content(
      "Button pressed with id: ${event.interaction.customId}")
  );
}

// Handling multiselect events is no different from handling button.
// Only thing that changes is type of function argument -- it now passes information
// about values selected with multiselect
Future<void> multiselectHandlerHandler(MultiselectInteractionEvent event) async {
  await event.acknowledge(); // ack the interaction so we can send response later

  // Send followup to button click with id of button
  await event.sendFollowup(MessageBuilder.content(
      "Options chosen with values: ${event.interaction.values}")
  );
}

void main() {
  final bot = NyxxFactory.createNyxxWebsocket("<TOKEN>", GatewayIntents.allUnprivileged);
  IInteractions.create(WebsocketInteractionBackend(bot))
    ..registerSlashCommand(singleCommand) // Register created before slash command
    ..registerButtonHandler("thisisid", buttonHandler) // register handler for button with id: thisisid
    ..registerMultiselectHandler("customId", multiselectHandlerHandler) // register handler for multiselect with id: customId
    ..syncOnReady(); // This is needed if you want to sync commands on bot startup.
}
```

<!-- prettier-ignore-start -->
[`nyxx`]: https://github.com/nyxx-discord/nyxx
[`nyxx_interactions`]: https://github.com/nyxx-discord/nyxx_interactions
[`MessageBuilder`]: https://pub.dev/documentation/nyxx/latest/nyxx/MessageBuilder-class.html
[`IInteractions`]: https://pub.dev/documentation/nyxx_interactions/latest/nyxx_interactions/IInteraction-class.html
[`ComponentMessageBuilder`]: https://pub.dev/documentation/nyxx_interactions/latest/nyxx_interactions/ComponentMessageBuilder-class.html
[`addComponent`]: https://pub.dev/documentation/nyxx_interactions/latest/nyxx_interactions/ComponentRowBuilder/addComponent.html
[`IInteractions.registerButtonHandler`]: https://pub.dev/documentation/nyxx_interactions/latest/nyxx_interactions/IInteractions/registerButtonHandler.html
[`IInteractions.registerMultiselectHandler`]: https://pub.dev/documentation/nyxx_interactions/latest/nyxx_interactions/IInteractions/registerMultiselectHandler.html
[`IInteractions.events.onMultiselectEvent`]: https://pub.dev/documentation/nyxx_interactions/latest/nyxx_interactions/IEventController/onMultiselectEvent.html
[`iinteractions.events.onButtonEvent`]: https://pub.dev/documentation/nyxx_interactions/latest/nyxx_interactions/IEventController/onButtonEvent.html
<!-- prettier-ignore-end -->
