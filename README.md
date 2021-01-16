## objd_gen

This package contains code generators made with source_gen. With annotations Widgets, Files, Packs and Projects can be written much more consisely.

### Installation

Include the package along with build_runner in your pubspec.yaml as a dev dependency:

```
dev_dependencies:
  build_runner:
  objd_gen: ^0.0.1
```

The generators put the new dart classes and functions in a new file alongside your annotated file. To make it available include it with the part statement:

```dart
import 'package:objd/core.dart';

part '[your_filename].g.dart';
```

After writing all your Widgets,this package generates the associated classes and functions with:

```
pub run build_runner build
```

Or if you want it to generate automatically after saving run:

```
pub run build_runner watch
```

### Widget

Writing a Widget becomes much simpler with the `@Wdg` annotation. You can just give it a function with needed parameters which returns a new Widget and the generators will figure out a Widget class to go along with it.

One simple case would be:

```dart
@Wdg
Widget helloWorld() => Log('Hello World!');
```

After running build_runner a new Widget **HelloWorld** is generated(inherited from the function name), so it is advisable to use lowercase functions.

Parameters also work like you exspect and you even get access to the widget context if you need to:

```dart
@Wdg
Widget helloName(String name, {String lastname = '', Context context}) => For.of([
  Comment('This was generated by HelloName on version ${context.version}'),
  Log('Hello $name $lastname!'),
]);
```

This would translate to a Widget in the following way, which you can use everywhere in your projects from now on:

```dart
class HelloName extends Widget {
  final String name;
  final String lastname;

  HelloName(
    this.name, {
    this.lastname = '',
  });

  @override
  Widget generate(Context context) => helloName(
        name,
        lastname: lastname,
        context: context,
      );
}
```

### File

Writing Minecrafts functions this style also becomes really easy. Just annotate a Widget variable that should be inside of your function with `@Func()`:

```dart
@Func()
final Widget load = HelloWorld();
```

This again would read the variable name `load` and generate a new variable called `LoadFile`, which includes the File Widget:

```dart
final File LoadFile = File(
  '/load',
  child: load,
);
```

Inside the parentheses of `@Func()` you can also provide various parameters to customize the file generation:

| @Func   |                                                                          |
| ------- | ------------------------------------------------------------------------ |
| name    | Provide a custom filename different from the variable name               |
| path    | Give a custom path for your function                                     |
| execute | whether to execute your File(when included somewhere in the widget tree) |
| create  | whether to actually create the file or not                               |

**Example:**

```dart
@Func(
  name: 'main',
  path: 'folder',
  execute: false,
  create: true,
)
final Widget main_widget = Comment('main file');
```

### Pack

The `@Pck()` annotation works similar to @Func. You annotate a File List variable and it generates a Widget for this Pack.

| @Pck |                           |
| ---- | ------------------------- |
| name | namespace of this pack    |
| main | path of the main function |
| load | path of the load function |

If you decide to not provide a namespace it again chooses your variable name.

**Example:**

```dart
@Pck(name: 'namespace', main: 'main', load: 'load')
final List<File> myPack = [
  LoadFile,
  MainFile,
];
```

This generates this widget:

```dart
class NamespacePack extends Widget {
  @override
  Widget generate(Context context) => Pack(
        name: 'namespace',
        files: myPack,
        load: File('load', create: false),
        main: File('main', create: false),
      );
}
```

### Project

And the last piece of creating a datapack is a `@Prj`. This can automatically generate a main function with all necessary pieces to actually generate all packs and files.

| @Prj        |                                                                                                   |
| ----------- | ------------------------------------------------------------------------------------------------- |
| name        | Name of the datapack                                                                              |
| version     | targeted minecraft version(as integer)                                                            |
| target      | directory to generate the datapack in                                                             |
| description | a description for the pack.mcdata                                                                 |
| genMain     | set this to false if you dont want a main function generated(will be generate\_[varname] instead) |

Once again we must annotate a Widget variable, that is the root of our widget tree:

```dart
@Prj(
  name: 'My awesome Datapack',
  target: './datapacks/',
  version: 17,
  description: 'A simple dp for demonstrating annotations',
)
final myProject = NamespacePack();
```

This generates the following in the `g.dart`:

```dart
void main([List<String> args = const []]) => createProject(
      Project(
        name: 'My awesome Datapack',
        target: './datapacks/',
        version: 17,
        description: 'A simple dp for demonstrating annotations',
        generate: myProject,
      ),
      args,
    );
```

### Full example

And thats it, you can now make an entire datapack in one objD file just with some functions and variables. Of course you can extend this principle to as many dart files as you like and make your datapack as complex and modular as you like.

You can see the full example [here](https://pub.dev/packages/objd_gen/example).

I hope these code generators help with making datapacks using objD. In case you encounter any problems or have any idea or feedback, feel free to reach out via [Discord](https://discord.gg/WvtCkyg) or [Email](mailto://contact@stevertus.com).
