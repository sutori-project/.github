# Sutori XML Guide - Spec 0.1-alpha

The various implementations of the Sutori expect a specific layout to the XML files you feed into it. This page explains the everything you need to know about that structure.



## Main Tag Layout
### The `<document>` tag.
The root element of every Sutori XML document should have a `<document>` tag. This tag is allowed to have the following singular children (commonly in this order):

- `<properties>` - Holds arbitary properties.
- `<resources>` - Holds multimedia resources (images etc...)
- `<actors>` - Holds actors that can be assigned to moments.
- `<moments>` - Holds the various moments in the dialog.

This tag can also have multiple of these tags in **load order**:

- `<include>` - Tells the document to load & join other Sutori XML documents.

Here is outline of how the document tag should look:

```xml
<document>
   <properties />
   <include after="false" />
   <resources />
   <actors />
   <moments />
   <include after="true" />
</document>
```



### The `<properties>` tag.
The properties tag allows you to specify document level properties. Every child tag within specifies the name of the property, and the contents acts as the value. Here are some common examples:

```xml
<properties>
   <width>800</width>
   <height>600</height>
   <title>Example Game</title>
</properties>
```

Note: attributes are ignored and not used in Sutori for this tag.



### The `<resources>` tag.
The resources tag is what is used to host the multimedia (images, audio etc...) that you wish to attach to the document. Heres an example of common values:

```xml
<resources>
   <image id="background-1" src="image/background1.png" />
   <image id="player-avatar" src="images/player.png" />
</resources>
```

Valid resources are current `<image>`, `<audio>` and `<video>`, and have available the following attributes:

- `id=""` - A unique ID that allows you to link to them in moments.
- `name=""` - A optional name, usually the filename/nickname.
- `src=""` - The path or URL to the multimedia (can also be a data uri).
- `preload="true/false"` - Weather or not to pre-load the resource.



### The `<actors>` tag.
This is an optional tag that you can use to give context to the moments in your dialog by assigning "actors". This is generally only seen if you are writing something like a game where this sort of context is important. Here's an example of how you'd use it:

```xml
<actors>
   <actor id="player" name="The Player" color="red" />
</actors>
```

Like resources, `<actor>` tags have available various attributes of note:

- `id=""` - A unique ID that allows you to link to them in moments.
- `name=""` - A optional display name.
- `color=""` - A hex color code to help identify an actor.


### The `<moments>` tag.
This is the most important child to the document tag. It will contain every moment of dialog for the document. Here's an example of how it should look:

```xml
<moments>
   <moment id="one" actor="player" goto="two">
      <text>You will not get away with this!</text>
   </moment>
   <moment id="two" actor="antagonist" goto="three">
      <text>Unfortunate for you, but I already have! HAHAHAHAHAHA!</text>
   </moment>
   <moment id="three" actor="player">
      <text>Noooooooooooooooooooo!</text>
   </moment>
</moments>
```

Note that there is no nesting, every `<moment>` tag appears sequentially.



### The `<moment>` tag.
This tag is used to describe a moment in the dialog. The best way to imagine it, is like a paragraph in a book. You can assign various XML attributes to a moment, here are the most significant:

- `id=""` - Sets a unique ID that allows Sutori to jump to it.
- `actor=""` - The ID of an actor that exists in the `<actors>` tag.
- `goto=""` - Tells Sutori which moment ID to goto next.
- `clear="true/false"` - Let engines know to clear the screen before displaying.

You can also add arbitary attributes to a moment tag too, in-engine they will be stored in a table property called "attributes".

Moment tags are also allowed to have a number of child elements. An is text, an image, a set of options, or even things like a trigger. Here is a list of allowed elements:

- `<text>` - A line of dialog text. 
- `<option>` - An option the user can select.
- `<media>` - Media to display/play.
- `<trigger>` - Fires an event trigger.
- `<set>` - Set a variable.
- `<load>` - Load more Sutori XML data (like `<include>` but encounter based).

These child elements have a very important attribute called `lang=""`. They specify weather they should get picked when a specific content culture/language is used in the engine. So:

```xml
<moment id="test" actor="player" custom-attrib="hello">
   
   <text lang="en-GB">Which door do you want to open?</text>
   <option lang="en-GB" target="door1">Door 1</option>
   <option lang="en-GB" target="door2">Door 2</option>
   
   <text lang="fr-FR">Quelle porte voulez-vous ouvrir?</text>
   <option lang="fr-FR" target="door1">Porte 1</option>
   <option lang="fr-FR" target="door2">Porte 2</option>
   
   <media lang="all" resource="background-1" for="background" />
   
</moment>
```

If the `en-GB` culture is enabled, only the first 3 elements (+ the image at the bottom) would be displayed to the player/user. And if `fr-FR` is enabled, only the last 4 elements would be presented.

If you don't specify a **lang** attribute, it's assumed that a document does not have support for multiple languages.




## Moment Elements
### The `<text>` tag.
This tag is used to contain language context aware text. In general it does not need anything other than a `lang=""` attribute.

Note: It's expected that you will only have 1 text tag per language & moment. However the engine implementations generally concatenate if multiple are found.

### The `<option>` tag.
This tag is used to offer an option to the end user. Like the `<text>` tag is also contains text, so a `lang=""` attribute is nessisary. Option tags have these futher optional attributes:

- `target=""` - The ID of the moment to goto if this option is chosen.
- `solver=""` - A hint to tell the engine how to understand the context of this option. Allowed values: `none`, `option_index`, `key_char_equality`, `text_equality`, or `custom`.
- `solver_callback=""` - If solver is set to `custom`, then this would be the name of a callback to implement a custom solver.

### The `<media>` tag.
This tag is used to attach media to a moment. It can be language sensitive, so a `lang=""` attribute is nessisary. Media tags have these futher optional attributes:

- `resource=""` - The ID of the loaded resource.
- `for=""` - A hint at what this media element should be used for.
- `coords=""` - Optional coordinates hint for the media.


### The `<trigger>` tag.
This tag is used to trigger an event of some sort. It can have a `lang=""` attribute, and typically has the following other attributes:

- `action=""` - The name of the action or callback to trigger.
- `body=""` - An optional single argument to pass to the trigger.#


### The `<set>` tag.
This tag is used to set a variable in the host game/app. It can have a `lang=""` attribute, but is typically ignored. The set tag has the following other attributes:

- `name=""` - The name of the variable to set.
- `value=""` - The new value to set.



## A Full Example
Here is a full example of every tag and attribute usable in Sutori:

```xml
<?xml version="1.0" encoding="utf-8"?>
<document xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema">
   <properties>
      <width>800</width>
      <height>600</height>
      <title>Example Game</title>
   </properties>
   <include>another_file.xml</include>
   <resources>
      <image id="background-1" src="image/background1.png" />
   </resources>
   <actors>
      <actor id="player" name="The Player" />
   </actors>
   <moments>

      <moment>
         <text data-example="test">Hi there!</text>
      </moment>

      <moment id="test" actor="player">
         <text>Which door do you want to open?</text>
         <option target="door1">Door 1</option>
         <option target="door2">Door 2</option>
         <text lang="fr-FR">Quelle porte voulez-vous ouvrir?</text>
         <option lang="fr-FR" target="door1">Porte 1</option>
         <option lang="fr-FR" target="door2">Porte 2</option>
         <media resource="background-1" for="background" />
      </moment>

      <moment id="door1" clear="true" goto="end">
         <text>You picked door1</text>
      </moment>

      <moment id="door2" clear="true" goto="end">
         <text>You picked door2</text>
      </moment>

      <moment id="end">
         <text>This is the end</text>
      </moment>
      
   </moments>
</document>
```


## Final Notes
Learning Sutori XML should be as easy as learning HTML. There is a strong layout to follow, and it's features allow a lot of flexibility.

For the best experience, we reccommend you check out Sutori Studio, as it's able to open, display and save files created using this format.
