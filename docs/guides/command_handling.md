---
title: Command handling
author: Rapougnac
timestamp: 2022-03-25
category: guides
sidebar_position: 8
---

# Command handling

Command handling is, by definition, a way to handle commands in multiple files.
Because by creating a lot of commands, your code in your main file will start to get really big and have a if/else if/switch hell.

Unlike many other languages, Dart does **not** have a way to dynamically — only known at runtime — import files.
But don't worry, there's still a way to handle properly your commands, slash commands and legacy commands.
[`nyxx_commands`] is a library that can handle commands for you.

## Creation of client

Let's setup a basic example, first, we need to create our client.

```dart
//highlight-next-line
import 'package:nyxx/nyxx.dart';

void main() {
  //highlight-start
  // nyxx_commands doesn't support INyxxRest yet, so we need to use INyxxWebsocket.
  final client = NyxxFactory.createNyxxWebsocket(
    '<TOKEN>',
    GatewayIntents.allUnprivileged | GatewayIntents.guildMembers,
  );
  //highlight-end
}
```

Now, by convenience, we register the basic plugins.
These aren't required for [`nyxx_commands`] to work

```dart
import 'package:nyxx/nyxx.dart';

void main() {
  // nyxx_commands doesn't support INyxxRest yet, so we need to use INyxxWebsocket.
  final client = NyxxFactory.createNyxxWebsocket(
    '<TOKEN>',
    GatewayIntents.allUnprivileged | GatewayIntents.guildMembers,
  );

  //highlight-start
  client
    ..registerPlugin(Logging())
    ..registerPlugin(CliIntegration())
    ..registerPlugin(IgnoreExceptions());
  //highlight-end
}
```

## Creating our [`CommandsPlugin`]

Ok, so now, we have client ready to be used, but we didn't used [`nyxx_commands`] after all. So let's get into it.

```dart
import 'package:nyxx/nyxx.dart';
//highlight-next-line
import 'package:nyxx_commands/nyxx_commands.dart';

void main() {
  // nyxx_commands doesn't support INyxxRest yet, so we need to use INyxxWebsocket.
  final client = NyxxFactory.createNyxxWebsocket(
    '<TOKEN>',
    GatewayIntents.allUnprivileged | GatewayIntents.guildMembers,
  );

  client
    ..registerPlugin(Logging())
    ..registerPlugin(CliIntegration())
    ..registerPlugin(IgnoreExceptions());

  //highlight-start
  // Next, we need to create our plugin. The plugin class used for nyxx_commands is `CommandsPlugin`
  // and we need to store it in a variable to be able to access it for registering commands and
  // converters.

  CommandsPlugin commands = CommandsPlugin(
    // The `prefix` parameter determines what prefix nyxx_commands will use for text commands.
    //
    // It isn't a simple string but a function that takes a single argument, an `IMessage`, and
    // returns a `String` indicating what prefix to use for that message. This allows you to have
    // different prefixes for different messages, for example you might want the bot to not require
    // a prefix when in direct messages. In that case, you might provide a function like this:
    // ```dart
    // prefix: (message) {
    //   if (message.startsWith('!')) {
    //     return '!';
    //   } else if (message.guild == null) {
    //     return '';
    //   }
    // }
    //```
    //
    // In this case, we always return `!` meaning that the prefix for all messages will be `!`.
    prefix: (message) => '!',

    // The `guild` parameter determines what guild slash commands will be registered to by default.
    //
    // This is useful for testing since registering slash commands globally can take up to an hour,
    // whereas registering commands for a single guild is instantaneous.
    //
    // If you aren't testing or want your commands to be registered globally, either omit this
    // parameter or set it to `null`.
    guild: Snowflake(Platform.environment['GUILD']!),

    // The `options` parameter allows you to specify additional configuration options for the
    // plugin.
    //
    // Generally you can just omit this parameter and use the defaults, but if you want to allow for
    // a specific behaviour you can include this parameter to change the default settings.
    //
    // In this case, we set the option `logErrors` to `true`, meaning errors that occur in commands
    // will be sent to the logger. If you have also added the `Logging` plugin to your client, these
    // will then be printed to your console.
    // `true` is actually the default for this option and was included here for the sake of example.
    options: CommandsOptions(
      logErrors: true,
    ),
  );

  // Next, we add the commands plugin to our client:
  client.registerPlugin(commands);


  // Finally, we tell the client to connect to Discord:
  client.connect();
  //highlight-end
}
```

## Registering a command

To register a command, we must first create an instance of the [`ChatCommand`] class.
This class represents a single command, slash command or not, and once added to the bot will
automatically be ready to use.

For our first command, let's create the famous `ping` command that'll reply with `pong`.

The first parameter is the command name.
Command names must obey certain rules, if they don't an error will be thrown.
Generally, using lower case letters and dashes (`-`) instead of spaces or undersoces will avoid any problems.

<br />
The second parameter is the command description. In traditional command frameworks,
command descriptions aren't required. However, Discord requires that all slash commands
have a description, so it is needed in nyxx_commands.
<br />
The third parameter is the function that will be executed when the command is ran.

The first parameter to this function must be a [`IChatContext`].

A [`IChatContext`] allows you to access various information about how the command was run: the user that executed it, the guild it was ran in and a few other useful pieces of information.
[`IChatContext`] also has a couple of methods that make it easier to respond to commands.
Since a ping command doesn't have any other arguments, we don't add any other parameters to the function.

```dart
// Finally, we tell the client to connect to Discord:
client.connect();

//highlight-start
ChatCommand ping = ChatCommand(
  'ping',
  "Checks if the bot's online",
  (IChatContext context) {
    // For a ping command, all we need to do is respond with `pong`.
    // To do that, we can use the `Context`'s `respond` method which responds to the command with
    // a message.
    context.respond(MessageBuilder.content('pong!'));
  },
);
//highlight-end
```

### Adding a command to the [`CommandsPlugin`]

Once we've created our command, we need to add it to our bot.

The commands on a bot can be represented with a parent-child tree that looks like this:

```
client
┗━ ping
```

```dart
ChatCommand ping = ChatCommand(
  'ping',
  "Checks if the bot's online",
  (IChatContext context) {
    // For a ping command, all we need to do is respond with `pong`.
    // To do that, we can use the `Context`'s `respond` method which responds to the command with
    // a message.
    context.respond(MessageBuilder.content('pong!'));
  },
);

//highlight-next-line
commands.addCommand(ping);
```

At this point, if you run this file, you should see a slash command named "ping" appear in
Discord. Executing it should make the bot send `pong!` to the channel the command was executed
in.
You can also send a text message starting with `!ping` and you should see a similar result.

<br />

If you don't want to be mentionned, you can use the [`mention`] option in the [`MessageChatContext`]'s [`respond`]
Like so:

```dart
//highlight-next-line
context.respond('pong!', mention: false);
```

<br />

:::info
You must check if the context is a [`MessageChatContext`] to apply the [`mention`] parameter.
:::

## Registering a command group

Command groups are a powerful tool that allow you to group commands together.
As an example, we'll create a command group `throw` with two sub-commands: `coin` and `die` (
the singular form of dice, not the verb).
Our command structure will look like this once we're done (the ping command we made earlier is
still there):

```
client
┗━ ping
┗━ throw
   ┗━ coin
   ┗━ die
```

Ok, so, let's create our first command group.

Similarly to [`ChatCommand`], the [`ChatGroup`] constructor's first two arguments are the group's name
and description.

However, there is no [`execute`] parameter; groups are not commands, and as such cannot be run,
so it makes no sense to have a function to execute. [`children`] is an optional parameter that allows you to specify what commands are part of this
group. It isn't the only way to add commands to a group though, so we'll add the `throw coin`
method here and the `throw die` command later.

```dart
//highlight-start
// We have to use the variable name `throwGroup` since `throw` is a reserved keyword. Note that
// the variable name does not change how the group behaves at all though.
ChatGroup throwGroup = ChatGroup(
  'throw',
  'Throw an object',
  children: [
    // We create a command in the same way as we created the `ping` command earlier.
    ChatCommand(
      'coin',
      'Throw a coin',
      (IChatContext context) {
        bool heads = Random().nextBool();

        context.respond(
          MessageBuilder.content('The coin landed on its ${heads ? 'head' : 'tail'}!'),
        );
      },
    )
  ],
);
//highlight-end
```

There's another way to add a command to a group. It's by using the [`ChatGroup`]'s [`addCommand`] method.

Like so:

```dart
//highlight-start
throwGroup.addCommand(
  ChatCommand(
    'die',
    'Throw a die',
    (IChatContext context) {
      int number = Random().nexInt(6) + 1;

      context.respond(MessageBuilder.content('The die landed on the $number!'));
    }
  ),
);
//highlight-end

// Finally, just like the `ping` command, we need to add our command group to the bot:
//highlight-next-line
commands.addCommand(throwGroup);
```

At this point, if you run this file, a new command should have appeared in the slash command menu on Discord: `throw`. Selecting it will let you choose from two sub-commands: `coin` or
`die`. Executing either of them should result in the bot sending a message with a random
outcome!

You can also send a text message starting with `!throw coin` or `!throw die` to execute the
commands. 


## Using command arguments

Command argumens are another powerful tool that allow you to get user input when using commands.
Adding arguments to your commands in nyxx_commands is simple, just add the argument as a
parameter to your [`execute`] function and nyxx_commands will do the rest, including:

- Converting the parameter name to a Discord-friendly name;
- Converting user input to the correct type;
- Inferring the best Discord slash command argument type to use for that argument

As an example, let's implement a `say` command that simply repeats what the user input.

```dart
ChatCommand say = ChatCommand(
  'say',
  'Make the bot say something',

  // As mentioned earlier, all we need to do to add an argument to our command is add it as a
  // parameter to our execute function. In this case, we take an argument called `message` and of
  // type `String`.
  (Context context, String message) {
    context.respond(MessageBuilder.content(message));
  },
);

// As usual, we need to register the command to our bot.
commands.addCommand(say);
```

At this point, if you run this file your command structure will look like this:

```
client
┗━ ping
┗━ throw
   ┗━ coin
   ┗━ die
┗━ say
```

A new command will also have appeared in Discord: `say`. Notice that this new command has an
argument, and Discord won't let you execute it unless your provide a value for that argument.

As usual, you can also execute the command with a message starting with `!say`.
However, if you try to run the command with a message that just says `!say`, you will notice an
error is thrown in your console. This is because you didn't provide a value for the `message`
argument.
Sending `!say hi` will result in the bot sending a message `hi` as expected.

:::caution Warning

Sending `!say hello, nyxx_commands!` will only make the bot send
a message with `hello,`. This is because the string parser assumes that one word = one
argument. To avoid this, put quotes around your argument to tell nyxx_commands that until the
next quote, all characters should be treated as a single argument (escaping quotes is
supported).
Running `!say "hello, nyxx_commands!"` results in `hello, nyxx_commands!` being sent as expected.

:::

## Using converters

Converters allow you to convert user input to specific Dart types. In the previous example, no conversion was needed as input received from Discord is already a [`String`].
However, the input received from Discord might not always be the type you need it to be. To
convert the input from Discord to the type needed for your [`execute`] function, the [`Converter`]
class is used.

[nyxx_commands] registers a few converters by default for commonly used types such as [`int`]s,
[`double`]s, [`IMember`]s and others. We'll look into creating custom converters later, and for now
just use the built-in converters.
Using converters is just as simple as using arguments: simply specify the argument name and
type, and [nyxx_commands] will determine which [`Converter`] to use for that argument.

As an example, let's create a `nick` command that changes a user's nickname.

```dart
ChatCommand nick = ChatCommand(
  'nick',
  "Change a user's nickname",
  // Setting the type of the `target` parameter to `IMember` will make nyxx_commands convert user
  // input to instances of `IMember`.
  (IChatContext context, IMember target, String newNick) async {
    try {
      await target.edit(nick: newNick);
    } on IHttpResponseError {
      context.respond(MessageBuilder.content("Couldn't change nickname :/"));
      return;
    }
    context.respond(MessageBuilder.content('Successfully changed nickname!'));
  },
);

commands.addCommand(nick);
```

At this point, if you run the file your command structure will look like this:

```
client
┗━ ping
┗━ throw
   ┗━ coin
   ┗━ die
┗━ say
┗━ nick
```

A new `nick` command will have been added to Discord's slash command menu. Selecting it will
prompt you for both a target and a new nickname.
Notice that the `target` argument will only allow you to select members of the guild the
command was ran in:
nyxx_commands automatically selected an appropriate slash command argument type for the
`target` parameter.

Also notice that the `newNick` argument name was changed to `new-nick`. nyxx_commands will
automatically convert your camelCase dart identifiers to Discord-compatible kebab-case argument
names, meaning you can keep your code clean without issues.

You can also run the command from a text message, with `!nick *target* *new-nick*`. Unlike
slash commands, there is no way to filter user input before it gets to our bot, so we might end
up with an invalid input.
If that is the case, the converter for [`IMember`] will be unable to convert the user input to a
valid member, and the command will fail with an exception.

:::info
Note that the bot must have the `MANAGE_MEMBERS` permission and have a higher role than the
target user for this command to succeed.
:::

## Using custom converters

Custom converters allow you to specify a custom way of converting user input to a specific type
that you can then use in your commands. You can define a custom converter in a few ways:

- By creating your own converter from scratch;
- By extending an existing converter;
- By combining existing converters

As an example, we will create converters for two enums defined at the bottom of this file:
`Shape` and `Dimension`.

### Using fully custom converters

To start off, we will create a converter for `Shape`s. We will need to create a brand new
converter from scratch for this, since no existing converter can be mapped to a `Shape`.

```dart
enum Shape {
  triangle,
  square,
  pentagon,
}


// Note that the variable is fully  typed, the typed generics on `Converter` are filled in.
// This allows nyxx_commands to know what the target type of this converter is.
Converter<Shape> shapeConverter = Converter<Shape>(
  // The first parameter to the `Converter` constructor is a function.
  // This function is what actually does the converting, and can return two things:
  // - An instance of the target type (in this case, `Shape`), indicating success;
  // - `null`, indicating failure (input cannot be parsed)
  //
  // The first parameter to the function is an instance of `StringView`. `StringView` allows you
  // to manipulate and extract data from a `String`, but also allows the next converter to know
  // where to start parsing its argument from.
  // The second parameter is the current `Context` in which the argument is being parsed.
  (view, context) {
    // In our case, we want to return a `Shape` based on the user's input. The `getQuotedWord()`
    // will get the next quoted word from the input.
    // Note that although `getWord()` exists, you shouldn't use it unless you have to as it can
    // mess up argument parsing later down the argument parsing.
    switch (view.getQuotedWord().toLowerCase()) {
      case 'triangle':
        return Shape.triangle;
      case 'square':
        return Shape.square;
      case 'pentagon':
        return Shape.pentagon;
      default:
        // The input didn't match anything we know, so we assume the input is invalid and return
        // `null` to indicate failure.
        return null;
    }
  },
  // The second parameter to the `Converter` constructor is a list (or any iterable) of
  // nyxx_interaction's `ArgChoiceBuilder`, allowing you to specify the choices that will be shown
  // to the user when running this command from a slash command.
  choices: [
    ArgChoiceBuilder('Triangle', 'triangle'),
    ArgChoiceBuilder('Square', 'square'),
    ArgChoiceBuilder('Pentagon', 'pentagon'),
  ],
);


// Once we've created our converter, we need to add it to the bot:
commands.addConverter(shapeConverter);
```

### Using assembled converters

For our `Dimension` converter, we can extend an existing converter: all `Dimension`s are
integers, so we can just extend the already existing `intConverter` and transform the integer
to a `Dimension`.

```dart
enum Dimension {
  twoD,
  threeD,
}

// To extend an existing `Converter`, we can use the `CombineConverter` class. This takes an
// exising converter and a function to transform the output of the original converter to the
// target type.
// Similarly to the shape converter, this variable has to be fully typed. The first type argument
// for `CombineConverter` is the target type of the inital `Converter`, and the second is the
// target type of the `CombineConverter`.
Converter<Dimension> dimensionConverter = CombineConverter<int, Dimension>(
  intConverter,
  // Also like `Converter`, the second parameter to the transform function is the current context.
  (number, context) {
    switch (number) {
      case 2:
        return Dimension.twoD;
      case 3:
        return Dimension.threeD;
      default:
        return null;
    }
  },
);
```

At this point, if you run the file you will see that the `favourite-shape` command has been
added to the slash command menu.
Selecting this command, you will be prompted to select a shape from the choices we outlined
earlier and a dimension. Note that in this case Discord isn't able to give us choices since we
haven't told it what dimensions are availible.

If you run the command, you will see that your input will automatically be converted to a
`Shape` and `Dimension` by using the converters we defined earlier.
Similarly to when we created the `nick` command, if either of our converters fail the command
will fail with an error.

## Using optional arguments

Optional argument allow you to allow the user to input something while providing a default
value.
Like converters, optional arguments are easy to use; just make the parameter in your [`execute`]
function optional.

You must use optional parameters and not named parameters in the [`execute`] function as the
order matters when using text commands. Note that when using slash commands, users can specify
optional arguments in any order and can omit certain optional arguments. nyxx_commands will fix
the ordering of the arguments, however you are not guaranteed to receive all optional arguments
up to the last one positionally.

As an example:

```dart
(IChatContext context, [String? a, String? b, String? c]) {}
```

In this case, `b` having a value does not guarantee `a` has a value. As such, it is always better to provide a default for your optional parameters instead of making them nullable.

```dart
// As an example for using optional arguments, let's create a command with an optional argument:
ChatCommand favouriteFruit = ChatCommand(
  'favourite-fruit',
  'Outputs your favourite fruit',
  (IChatContext context, [String favourite = 'apple']) {
    context.respond(MessageBuilder.content('Your favourite fruit is $favourite!'));
  },
);

commands.addCommand(favouriteFruit);
```

<br />

At this point, if you run the file you will be able to use the `favourite-fruit` command. Once
you've selected the command in the slash command menu, you'll be given an option to provide a
value for the `favourite` argument.
If you don't specify a value for the argument, the default value of `'apple'` will be used. If
you do specify a value, that value will be used instead.

Using optional arguments in text commands works as expected: arguments are parsed from the user
input until there is no text left to parse or all possible arguments. This means that if you
want to specify an optional argument, you just have to type it in as you would with a
non-optional argument and the arguments will be filled in left to right in the order they were
declared.

## Using checks

Command checks allow you to restrict a command's usage. There are a few built-in checks that
integrate with Discord's slash command permissions, and a special cooldown check.

As an example, we'll create a command with a cooldown:

```dart
ChatCommand alphabet = ChatCommand(
  'alphabet',
  'Outputs the alphabet',
  (IChatContext context) {
    context.respond(MessageBuilder.content('ABCDEFGHIJKLMNOPQRSTUVWXYZ'));
  },
  // Since this command is spammy, we can use a cooldown to restrict it's usage:
  checks: [
    CooldownCheck(
      CooldownType.user,
      Duration(seconds: 30),
    )
  ],
);

commands.addCommand(alphabet);
```

## Using converters override

Converter overrides allow you to override which [`Converter`] is used to convert a specific argument.
They can be used, for example, to filter the output of a converter so that only certain
instances are ever passed to the function.
You might have noticed that if we run the command `!say ""`, the bot will throw an error
because it cannot send an empty message. This is because the string [`Converter`] saw `""` and
interpreted it as an empty string - which the bot then tried to send.

To counter this, let's create a converter that only allows non-empty strings:

```dart
// Since the converter is going to be passed into a decorator, it must be `const`. As such, any
// functions used must either be static methods or top-level functions.
// You can see the implementation of `filterInput` at the bottom of this file.
const Converter<String> nonEmptyStringConverter = CombineConverter(stringConverter, filterInput);

ChatCommand betterSay = ChatCommand(
  'better-say',
  'A better version of the say command',
  (
    IChatContext context,
    @UseConverter(nonEmptyStringConverter) String input,
  ) {
    context.respond(MessageBuilder.content(input));
  },
);

commands.addCommand(betterSay);

String? filterInput(String input, Context context) {
  if (input.isNotEmpty) {
    return input;
  }
  return null;
}
```

At this point, if you run the file, a new command `better-say` will have been added to the bot.
Attempting to invoke it with an empty string (`!better-say ""`) will cause the argument to
fail parsing.

That's all! Well done, you've reached the end of this guide. You're now able to go and create your own bot on your side.
Remember; You just need to create a function or a class in a specific file, importing it, and register the function.

<!-- Prettier format in lower-case named hidden links, so we need to ignore this -->
<!-- prettier-ignore-start -->
[`nyxx_commands`]: https://github.com/nyxx-discord/nyxx_commands/tree/main
[nyxx_commands]: https://github.com/nyxx-discord/nyxx_commands/tree/main
[`ChatCommand`]: https://pub.dev/documentation/nyxx_commands/latest/nyxx_commands/ChatCommand-class.html
[`ChatGroup`]: https://pub.dev/documentation/nyxx_commands/latest/nyxx_commands/ChatGroup-class.html
[`IChatContext`]: https://pub.dev/documentation/nyxx_commands/latest/nyxx_commands/IChatContext-class.html
[`execute`]: https://pub.dev/documentation/nyxx_commands/latest/nyxx_commands/ChatCommand/execute.html
[`children`]: https://pub.dev/documentation/nyxx_commands/latest/nyxx_commands/ChatGroup/children.html
[`addCommand`]: https://pub.dev/documentation/nyxx_commands/latest/nyxx_commands/ChatGroup/addCommand.html
[`MessageChatContext`]: https://pub.dev/documentation/nyxx_commands/latest/nyxx_commands/MessageChatContext-class.html
[`respond`]: https://pub.dev/documentation/nyxx_commands/latest/nyxx_commands/MessageChatContext/respond.html
[`mention`]: https://pub.dev/documentation/nyxx_commands/latest/nyxx_commands/MessageChatContext/respond.html
[`String`]: https://api.dart.dev/stable/2.16.2/dart-core/String-class.html
[`double`]: https://api.dart.dev/stable/2.16.2/dart-core/double-class.html
[`int`]: https://api.dart.dev/stable/2.16.2/dart-core/int-class.html
[`IMember`]: https://pub.dev/documentation/nyxx/latest/nyxx/IMember-class.html
[`Converter`]: https://pub.dev/documentation/nyxx_commands/latest/nyxx_commands/Converter-class.html
[`CommandsPlugin`]: https://pub.dev/documentation/nyxx_commands/latest/nyxx_commands/CommandsPlugin-class.html
<!-- prettier-ignore-end -->
