# Fate Syntax

Doc Version 1.3
Fate Version 466
p4p

This doc describes how to write source code for Fate games.



## Files

Fate source code is written in a special syntax, and saved in a text file with the `.fate` extension. A game can be written in as many fate files as necessary. The order of files does not matter but each file should have a unique name. `.fate` files can exist in any sub directory of the `source` directory, which should be placed in the top level game documents directory specified by the player at the start of the game.

```
My Game Title
-> /source
---> my file.fate
---> /my folder
-----> my nested file.fate
```



## Headers

Headers are the core building blocks of a Fate game. They have a general form of `:: Title [Tag]`. There are two types of headers, passages and rules.



### Passages

A passage is a simple header that groups text to display with commands.

```
:: Working with Fate
Here is some text to go in the passage.
It's not very exciting though.
```

`Working with Fate` is the title of that passage. The title acts as the identifier of the passage, allowing links and other commands to access the passage by name.
A link to the previous example passage would use the title like this`[[Working with Fate]]`. Links are a special kind of command discussed later, but the important thing to know is that passages are linked to explicitly by name.



####  Start

A Fate game has to begin somewhere, so we need to define a special passage titled `Start`.
```
:: Start
This is the entry point of the game.
```



### Rules

Rules are headers just like passages except they are linked to implicitly rather than explicitly.
```
:: A rule [rule($R == 22)]
This is the text of a rule :)
```

That example creates a rule titled `A rule`. Unlike passages, the titles of rules aren't used to access the rule. Instead a numerical identifier is used and it is specified in the tag section of the header. In this example we call the function `rule($R == 22)` within the tag.

The `rule()` function marks that we are writing a rule header not a passage header. The arguments between the braces `(` and `)` are the criteria for the rule. There may be many rules within a fate game. To access the correct rule, criteria must be set for each rule to determine when it should be selected for use. These criteria must be true for the rule to match otherwise the rule is excluded.

`$R == 22` is a conditional statement that states: the variable `$R` must be equal to 22. `$R` is a special variable used when defining and accessing rules. As an example, if the player were to click an implicit link Fate does a search of all rules that we've defined. If the implicit link sets the value of `$R` to 22, they may get back the example rule we defined above.

The usefulness of this is that when making a game in Fate we don't need to know exactly where a link will take the player. Also the danger of this is that we don't know exactly where a link will take the player.



####  More specific Rules

Using the `$R` variable to link to rules allows us to add variation to our game indirectly. Consider the following rules
```
:: Banana is the best [rule($R == 22)]
My fav fruit is a banana.

:: Oranges are the best [rule($R == 22)]
My fav fruit is an orange.
```

Both of these rules include the criteria `$R == 22`. Can you guess which Fate will choose to present?

If you said the first you are right. If you said the second you are right too. By default rule selection is random across equal rules. We can use this to add variation to our game by simply adding more rules.

Rules are no longer equal once they have a different amount of criteria.

For example, as the player does things in the game we can set variables for anything we like. These variables can also be used in the criteria of a rule definition.
```
:: Kiwi is the best [rule($R == 22)]
My fav fruit is kiwi.

:: Banana is the best [rule($R == 22 && $lovesBananas == true)]
My fav fruit is a banana.

:: Oranges are the best [rule($R == 22 && $lovesOranges == true)]
My fav fruit is an orange.
```


All three rules are looking for `$R == 22`, but notice that the second rule includes `$lovesBananas == true` and the third requires `$lovesOranges == true`. These rules are heavier than the first, meaning they have more criteria and are more specific. More specific rules are more relevant to the player, so they will always be selected before lower criteria rules so long as their criteria are tested to be true at run time.

One of these more specific rules will be selected before the first rule, since they both have more criteria and are thus more relevant to the player's previous actions in the game. It is important to note that we still include a default rule with only `$R == 22` since the more specific rules may never apply. Having a fallback rule is very important to avoid dead ends.


Lets look at a different situation
```
:: Kiwi is the best [rule($R == 22)]
My fav fruit is kiwi.

:: Banana is the best [rule($R == 22 && $hatesKiwi == true)]
My fav fruit is a banana.

:: Oranges are the best [rule($R == 22 && $hatesKiwi == true)]
My fav fruit is an orange.
```

The second and third rules now require `$hatesKiwi == true`. In this case if they player dislikes kiwis, Fate will choose bananas or oranges for the player.



####  Rule Shorthand

For simple rules (rules that only have a `$R` equality criterion) it is possible to skip the full criteria declaration and instead only pass the number value to the rule function

```
:: A rule [rule(22)]
Rules act just like passages, except they are linked to implicitly rather than explicitly.
```



#### Unique Rules

There are a few unique rules that serve special purposes.



##### Pre

The `^pre` rule is called before the processing of a passage or rule. It is helpful when you want to include content that should always show above the passage text, like stats displays.

```
:: PRE [rule(^pre)]
TODO
```



##### Post

`^post` rule is called after the processing of a passage or rule. It can tack on additional content that should always show after a passage.

```
:: POST [rule(^post)]
TODO
```



## Links

Links connect passages and rules into a game.



### Simple links

Here is a simple link example
```
:: Start
This is the entry point of the game.
[[Linking between passages]]


:: Linking between passages
A simple link uses the title of a passage as the text to display to the player. Neat.
```

A link is always defined on a new line starting with `[[` and ending with `]]`. Basic links like `[[Linking between passages]]` use the title of a passage as the text of the link.

Note that despite the position of a link within the passage or rule body, it will always be presented below all text output to the player.



### Complex links

Complex links allow us to further customize what happens when a link is clicked.

Here is an example of a complex link

```
:: Start
This is the entry point of the game.
[["Link Text", to(Linking between passages)]]


:: Linking between passages
A simple link uses the title of a passage as the text to display to the player.
```

This link still takes the player to the passage `Linking between passages`, but the text of the link will be `Link Text` as it appears within the quotes. Notice the arguments of the link (the link text and to function) are separated by a comma.



#### Complex link To function

The to() function allows us to create implicit links
```
:: Start
This is the entry point of the game.
[["Link Text", to(22)]]

:: A rule [rule($R == 22)]
Rules act just like passages, except they are linked to implicitly rather than explicitly.
```

Here the `$R` key will be set to the value `22` once the link is clicked by the player. Because the rule `A rule` has the criteria `$R == 22` it will be selected as a potential rule during the game.



#### Complex link Set function

We can set variables within a complex link

```
:: Start
This is the entry point of the game.
[["Link Text", to(22), set($myKey = 123)]]
```

This link sets the value of the variable `$myKey` to 123 when clicked.



#### Complex link If function

The If function lets us add criteria to a link. These criteria must then be true for the link to be displayed to the player.

```
:: Start
This is the entry point of the game.
[["Link Text", to(^MyRule), if($myKey == 123)]]
```



## Variables

Variables in fate are alpha-numeric text prefixed with a dollar sign, `$`. Variables are always public and global values. They can be presented to the player and used in criteria across commands and structures.



### Reserved Variables

The following variable names are reserved

Variable | Use
---------| ------------------------------------------
$R       | used in the definition and access of Rules



## Criteria

Criteria used in rules, links, etc. take the following format
```
$R == ^myKey
```

The value on the right side must be a number, named number, or a variable. Here the variable `$R` is compared to the named number `^myKey` 

Note: Undefined variables used in a criterion will cause the criterion to be false.


The following are valid comparators when building criteria

Operator | Example       | Description
-------- | ------------- | -----------
==       | $myKey == 123 | Equal to
!=       | $myKey != 123 | Not equal to
>        | $myKey > 123  | Greater than
>=       | $myKey >=123  | Greater than equal to
<        | $myKey < 123  | Less than
<=       | $myKey <= 123 | Less than equal to
%        | $myKey % 123  | Divisible by
!%       | $myKey !% 123 | Indivisible by



### Multiple Criteria

Multiple criteria are separated with `&&`

```
$R == ^MyRule && $myKey == ^MyRule
```

Note: It is not possible to use the logical or operator `||` in criteria.



## Commands

Commands enable special functionality within a passage or rule. Because they are different from the free text of a passage or rule, commands must be declared on a new line somewhere after the passage/rule header.

Commands are always marked with `>` after a new line.



### Rule Commands

There are two commands that modify Rule selection in Fate, priority and weight. The criteria command is a shorthand for defining multiple rules under a single set of base criteria.



#### Criteria

The criteria command is a helpful shorthand to create multiple rules under a single Rule header:

```
:: My rule with a long title for no reason [rule($R == 22)]
Rules act just like passages, except they are linked to implicitly rather than explicitly.

> criteria $myKey == 123
We can use the >criteria command to conveniently place variations of a rule under a single rule declaration.
```

is equivalent to

```
:: My rule with a long title for no reason [rule($R == 22)]
Rules act just like passages, except they are linked to implicitly rather than explicitly.

:: My rule with a long title for no reason [rule($R == 22 && $myKey == 123)]
We can use the >criteria command to conveniently place variations of a rule under a single rule declaration.
```

Note: Each of the rules specified by the criteria command requires its own text, commands, etc. as if it were declared separately.



#### Priority

The priority command lets us set priorities for rules. Consider the example
```
:: Kiwi is the best [rule($R == 22)]
My fav fruit is kiwi.

:: Banana is the best [rule($R == 22 && $lovesBananas == true)]
My fav fruit is a banana.

:: Oranges are the best [rule($R == 22 && $lovesOranges == true)]
My fav fruit is an orange.
```

If we wanted the first rule `Kiwi is the best` to be selected over the other rules even though they are heavier (first to be selected), we could give rule 1 a higher priority.

```
:: Kiwi is the best [rule($R == 22)]
My fav fruit is kiwi.
> priority 2

:: Banana is the best [rule($R == 22 && $lovesBananas == true)]
My fav fruit is a banana.

:: Oranges are the best [rule($R == 22 && $lovesOranges == true)]
My fav fruit is an orange.
```

`> priority 2` gives the first rule a priority of 2, greater than the default value of 1 on the other rules. If the `Kiwi is the best` rule fails the criteria check, Fate will still check lower priority rules.



#### Weight

The weight command lets us use numerical weights to increase the odds of fate selecting one rule above others of a similar criteria count and priority. Consider the following
```
:: Kiwi is the best [rule($R == 4)]
My fav fruit is kiwi.

:: Banana is the best [rule($R == 4 && $lovesBananas == true)]
My fav fruit is a banana.
> weight 1000

:: Oranges are the best [rule($R == 4 && $lovesOranges == true)]
My fav fruit is an orange.
> weight 50
```
Here if we assume that `$lovesBananas` is true, then fate will decide between the first and second rules randomly. Part of the random selection process takes into account weighting so that rule two with a weight of 1000 is more likely to be selected than rule three.



### Variable Commands



#### Set Command

We can set variables using the set command.



##### Setting numbers

Setting a number variable
```
:: Start
This is the entry point of the game.
> set $myKey = 1000
```
Here, the variable $myKey is assigned the value of 1000. Note that we can also use floating point numbers.

We can change the value of the key at any time with another set call
```
:: Start
This is the entry point of the game.
> set $myKey = 1000
> set $myKey = 10
```



##### Simple math operations

We can perform simple math on a variable
```
:: My Passage
> set $health += 50
You pick up the health potion and feel better
```
Operator | Example      | Description
-------- | ------------ | -----------
=        | $keyA = 123  | Assign a variable
>=       | $keyA >= 123 | Assign the greater of a variable and an operand
<=       | $keyB <= 123 | Assign the lesser of a variable and an operand
+=       | $keyC += 123 | Add to a variable
-=       | $keyD -= 123 | Subtract from a variable
*=       | $keyE *= 123 | Multiple a variable
/=       | $keyF /= 123 | Divide a variable (can't be zero)
%=       | $keyF %= 123 | Divide a variable (can't be zero) and set to remainder
%+       | $keyG %+ 123 | Add fairly to a variable [ChoiceScript](http://choicescriptdev.wikia.com/wiki/Arithmetic_operators)
%-       | $keyH %- 123 | Subtract fairly from a variable [ChoiceScript](http://choicescriptdev.wikia.com/wiki/Arithmetic_operators)



##### Setting text

Setting a text variable
```
:: Start
This is the entry point of the game.
> set $myKey = "Sample Text"
```
Here, the variable $myKey is assigned the value of Sample Text. Notice that the words are wrapped in quotes.



##### Text operations

Operator | example				| Description
-------- | -------------------- | -----------
=		 | $keySA = "any text"  | Assign a variable
>=		 | $keySA >= "any text" | Assign the longer of a variable and an operand
<=		 | $keySB <= "any text" | Assign the shorter of a variable and an operand
+=		 | $keySC += "any text" | Append text to the variable
-=		 | $keySD -= "any text" | Find and remove text from the variable



##### Multiple assignments

We can put multiple statements in a single set command call.
```
:: Start
This is the entry point of the game.
> set $myKey = 1000; $mySecondKey = ^anything; $myThirdKey = "Sample Text"
```



##### Math operations

You can use math operations to modify/generate numerical values

```
> set $key = abs
> set $key = ceiling
> set $key = floor
> set $key = random
> set $key = randomPercent
> set $key = range(123, 123)
> set $key = rangeWhole(123, 123)
> set $key = round
```

Operator      | example				       | Description
------------- | -------------------------- | -----------
abs           | > set $key = abs           | Assign absolute value
ceiling       | > set $key = ceiling       | Assign largest number greater than or equal
floor         | > set $key = floor         | Assign largest number lesser than or equal
random        | > set $key = random        | Assigns a floating point value between 0.0 and 1.0
randomPercent | > set $key = randomPercent | Assigns random number between 1-100 to a variable
range         |                     	   | Assign a value between two floating point numbers
rangeWhole    |                     	   | Assign a value between two integer numbers
round         | > set $key = round         | Assign to the closest integer value



### Visual Commands

Visual commands allow us to present interactive content like input fields and toggles, line separators, images, etc. Lets see how much of that I implement. -_-



#### Display Command

The display function allows you to nest passages and rules. This is useful in preventing duplication of text.



##### Explicit Display Command

To display an explicit passage, wrap the title of the passage with opening and closing `:` 

```
:: Start
This is the entry point of the game.
> display :My Passage:
[[Another Passage]]


:: My Passage
The rest of the game continues here.
[[Some Other Passage]]
```

That example is equivalent to the following

```
:: Start
This is the entry point of the game.
The rest of the game continues here.
[[Another Passage]]
[[Some Other Passage]]
```

Notice that the text portions of the passage merge. Keep in mind that this merge works by adding the content on a new line rather than the same line, so all of the visual settings in the second passage remain the same. All of the links from the displayed passage are copied as well as any other commands. Furthermore, the displayed passage can also display passages, creating multiple levels of passage content in the final result.



##### Implicit Display Command

The display command can work explicitly with the name of a passage, or implicitly with a given R value.
```
:: Start
This is the entry point of the game.
> display ^myRule
[[Another Passage]]


:: My Displayed Rule [rule(^myRule)]
The rest of the game continues here.
[[Some Other Passage]]
```


#### Run Command

The run command will apply the commands of a rule or passage, without printing anything. This is useful for applying calculations without concerns about white space or unintended output text.


##### Explicit Run Command

```
:: Start
This is the entry point of the game.
> run :Some Calculations To Do:
Here is some text.


:: Some Calculations To Do
> set $myKey = 1000
```



##### Implicit Run Command

```
:: Start
This is the entry point of the game.
> run ^myRule
Here is some text.


:: My Runnable Rule [rule(^myRule)]
> set $myKey = 1000
```



#### Input Command

The input command lets the player input text, numbers and values directly into an editable text area. A text area can accept the following types of input:
* text
* integer
* decimal
* alphanumeric
* email
* name

The input command accepts the type, key that will take the input, and a label that will act as a placeholder in unset keys.
```
> input alphanumeric, $myKey, "placeholder text"
> input decimal, $myKey, "placeholder text"
> input email, $myKey, "placeholder text"
> input integer, $myKey, "placeholder text"
> input name, $myKey, "placeholder text"
> input text, $myKey, "placeholder text"
```
To set a default value for the input key, set the key prior to calling the input command.



#### Toggle Command

A toggle command gives the player a check box to tick to set a value to true or false.
```
> toggle $myKey, "label text"
```
To set a default value for the input key, set the key prior to calling the toggle command.



##### Toggle Group Command

You can use a toggle to present the player with several options that set a key to a numerical value. Toggle groups are defined just like toggles with the addition of pairs of labels and numerical values.
```
> toggle $myKey, "label text", "first option label", ^myValue, "second option label", ^myOtherValue
```



## Free Text

Free text is just text that appears after a rule or passage definition.



### Embed

Embeds are a useful way of marking spans of text as special in some way. Embeds can exist anywhere within free text. They begin with `[` and end with `]`.



#### Embed Text

Each embed needs text to display. This text can be wrapped in quotes.

```
:: Start
This is some text along with ["my text embed"].
```

To create more dynamic content, variables and plain text passages and rules can provide the text for an embed.



##### Embed Print Variables

We can output the value using the print embed function.
```
:: Start
This is the entry point of the game.
> set $myKey = "Sample Text"
[[Another Passage]]

:: Another Passage
The value of $myKey is [$myKey].
```

Here the print statement `[$myKey]` in `Another Passage` would be replaced with `Sample Text`.



##### Embed Print Passage

For simple passages
```
:: Start
This is the entry point of the [:My Passage:]
```



##### Embed Print Rule

For simple rules
```
:: Start
This is the entry point of the [^MyRule]
```

A number will also work
```
:: Start
This is the entry point of the [1000]
```



#### Embed If

if within an embed statement will cause the embed to not be displayed if the conditions specified resolve to false.
```
:: Start
This is the entry [$point, if($myKey == 123)] of the game.
```



##### Embed Mutation

The text content of an embed can be changed using mutation keywords.


##### Embed Mutation Plural
This mutation turns the content of the embed Plural.
```
◊ Make plural "dog" -> ["dog", plural] 'dogs'
◊ Make plural "beach" -> ["beach", plural] 'beaches'
◊ Make plural "person" -> ["person", plural] 'people'
```

##### Embed Mutation Singular
This mutation turns the content of the embed Singular.
```
◊ Make singular "dogs" -> ["dogs", singular] 'dog'
◊ Make singular "beaches" -> ["beaches", singular 'beach'
◊ Make singular "people" -> ["people", singular 'person'
```


##### Embed Mutation Lower Case
This mutation turns the content of the embed to Lower Case.
```
◊ Make lowercase "HERE GOES NOTHING" -> ["HERE GOES NOTHING", lower]
```
##### Embed Mutation Sentence Case
This mutation turns the content of the embed to Sentence Case. 'My text'
```
◊ Make sentence "MyText" -> ["myText", sentence]
◊ Make sentence "myText" -> ["myText", sentence]
◊ Make sentence "my_text" -> ["my_text", sentence]
◊ Make sentence "my text" -> ["my_text", sentence]
◊ Make sentence "MY TEXT" -> ["my_text", sentence]
```
##### Embed Mutation Title Case
This mutation turns the content of the embed to Title Case. 'My Text'
```
◊ Make title "MyText" -> ["myText", title]
◊ Make title "myText" -> ["myText", title]
◊ Make title "my_text" -> ["my_text", title]
◊ Make title "my text" -> ["my_text", title]
◊ Make title "MY TEXT" -> ["my_text", title]
```
##### Embed Mutation Upper Case
This mutation turns the content of the embed to Upper Case.
```
◊ Make uppercase "here goes nothing" -> ["here goes nothing", upper]
```


##### Embed Mutation Camel Case
This mutation turns the content of the embed to Camel Case. 'camelCase'
```
◊ Make camel "MyText" -> ["myText", camel]
◊ Make camel "my_text" -> ["my_text", camel]
◊ Make camel "my text" -> ["my_text", camel]
◊ Make camel "MY TEXT" -> ["my_text", camel]
```
##### Embed Mutation Pascal Case
This mutation turns the content of the embed to Pascal Case. 'PascalCase'
```
◊ Make pascal "myText" -> ["myText", pascal]
◊ Make pascal "my_text" -> ["my_text", pascal]
◊ Make pascal "my text" -> ["my_text", pascal]
◊ Make pascal "MY TEXT" -> ["my_text", pascal]
```


##### Embed Mutation Hyphenate
This mutation Hyphenates the content of the embed.
```
◊ Make hyphenate "my_text" -> ["my_text", hyphenate]
◊ Make hyphenate "my text" -> ["my_text", hyphenate]
◊ Make hyphenate "MY TEXT" -> ["my_text", hyphenate]
```
##### Embed Mutation Underscore
This mutation Underscores the content of the embed.
```
◊ Make underscore "HERE GOES NOTHING" -> ["HERE GOES NOTHING", underscore]
```


##### Embed Mutation Dehumanize
This mutation dehumanizes the content of the embed. 'MyText'
```
◊ Make dehumanize "myText" -> ["myText", dehumanize]
◊ Make dehumanize "my_text" -> ["my_text", dehumanize]
◊ Make dehumanize "my text" -> ["my_text", dehumanize]
◊ Make dehumanize "MY TEXT" -> ["my_text", dehumanize]
```
##### Embed Mutation Humanize
This mutation humanizes the content of the embed. 'my text'
```
◊ Make humanize "MyText" -> ["myText", humanize]
◊ Make humanize "myText" -> ["myText", humanize]
◊ Make humanize "my_text" -> ["my_text", humanize]
◊ Make humanize "my text" -> ["my_text", humanize]
◊ Make humanize "MY TEXT" -> ["my_text", humanize]
```


##### Embed Mutation Ordinal
This mutation turns a number into an ordinal string used to denote the position in an ordered sequence such as 1st, 2nd, 3rd, 4th.
```
◊ Make ordinal "41" -> ["41", ordinal] 'forty-first'
◊ Make ordinal "3" -> ["3", ordinal] 'third'
◊ Make ordinal "421" -> ["421", ordinal] 'four hundred and twenty-first'
◊ Make ordinal "5152" -> ["5152", ordinal] 'five thousand one hundred and fifty-second'
◊ Make ordinal "65152" -> ["65152", ordinal] 'sixty-five thousand one hundred and fifty-second'
◊ Make ordinal "0" -> ["0", ordinal] 'zeroth'
◊ Make ordinal "1" -> ["1", ordinal] 'first'
◊ Make ordinal "10" -> ["10", ordinal] 'tenth'
◊ Make ordinal "1000" -> ["1000", ordinal] 'thousandth'
◊ Make ordinal "10000" -> ["10000", ordinal] 'ten thousandth'
◊ Make ordinal "100000" -> ["100000", ordinal] 'hundred thousandth'
◊ Make ordinal "1000000" -> ["1000000", ordinal] 'millionth'
```



## Comments

Comments exclude text from compilation. The are defined with an opening `/*` and closing `*/`
```
:: Working with Fate
Here is some text to go in the passage.
It's not very exciting though./* This is a comment */
```
In that example the text `/* This is a comment */` would not appear in the game.



## Metadata

Every game has associated metadata that can be declared in the source code. Fate uses a special syntax to declare a piece of game metadata
```
@author p4p @end
```
Metadata declarations can appear anywhere in any fate file, but it is a good idea to place them at the top of a source file for clarity.



### Single call metadata

The following metadata calls only need to be used once.



#### Author

The primary author/group behind the game.
```
@author p4p @end
```



#### Brief Description

A short (140 character) description of the game.
```
@brief The best of games, or the worst of games. @end
```



#### Content Rating

A rating title for the game. See [ESRB](https://www.esrb.org/ratings/ratings_guide.aspx)
```
@contentRating EVERYONE 10+ @end
```



#### Credits

A full listing of credits for the game. Note that this does not include any computed information from `@author` or `@contributor`.
```
@credits 
Created by p4p
Lack of artwork p4p
Lack of sound p4p
@end
```



#### Description

A full description of the game.
```
@description 
The game

This game is about some stuff I guess.
fin
@end
```



#### Identifier

A unique identifier for the game in reverse domain form.
```
@identifier com.Bunkville.Demo @end
```



#### License

A license declaration for the game
```
@license 
© 2016 p4p. All rights reserved.
@end
```



#### Sensitive Content

A special flag that marks the game as having content that requires an age disclaimer before play.
```
@sensitiveContent true @end
```



#### Title

The full title of the game.
```
@title Demo Game @end
```



#### Updates

A full update list for the game.
```
@updates 
v1.0.3
	Added some text
	Removed some other text
v1.0.2
	Made some stuff up
v1.0.1
	Got the thing working
@end
```



#### Version

The version of the game.
```
@version v1.0.3 @end
```



### Multiple call metadata

The following metadata calls can be used as many times as necessary.



#### ContentDescriptor

Content descriptors 'indicate content that may have triggered a particular rating and/or may be of interest or concern' ... see [ESRB](https://www.esrb.org/ratings/ratings_guide.aspx)
```
@contentDescriptor Drug Reference - Reference to and/or images of illegal drugs @end
```

Note that we can repeat the same `@contentDescriptor` call across files and it will only appear once in the initial content warning for the game.

Note 2, it may be wise to place content that requires a `@contentDescriptor` call in a unique file so that the player can simply remove the file from the game if they don't want that content. This will require using Rules so that the game can fall back to non offending content.



#### Phobia

Phobias or triggers 'indicate whether or not' a game contains 'content that could be potentially harmful, traumatic, or even triggering for the audience' ... see [Game Phobias](http://www.gamephobias.com/index.php?title=Content_Advisory_Tags)
```
@phobia 
Needles

        Games tagged with this content contain instances in which an individual is subjected to, or even merely present in situations in which hypodermic needles are present. 
@end
```

Note that we can repeat the same `@phobia` call across files and it will only appear once in the initial content warning for the game.

Note 2, it may be wise to place content that requires a `@phobia` call in a unique file so that the player can simply remove the file from the game if they don't want that content. This will require using Rules so that the game can fall back to non offending content.



#### Contributor

When contributing to a game you can use an `@contributor` call to add yourself as a contributor. In the about interface of the game, contributors are displayed in order of contributions made (the time `@contributor` was used for each unique name).
```
@contributor p4p @end
```



## Named Numbers

As a convenience it is possible to use a text name in place of a number value in several places throughout the source of a game. It is done with the syntax `^myNumber` where `^` prefixes a text value for the unique number. The actual number used is unspecified except for `^true` and `^yes` which equal 1, and `^false` and `^no` which equal 0.

Here is an example rule using a named number in place of the `$R` value
```
:: A rule [rule($R == ^MyRule)]
Rules act just like passages, except they are linked to implicitly rather than explicitly.
```



### Reserved Named Number

The following Named Numbers are used by the system and should be used carefully.


Named Number | Value
------------ | -----
^no          | 0
^false       | 0
^yes         | 1
^true        | 1
^pre         | 5
^post        | 6