---
layout: post
title: "Optimizing Sublime Text for Development in Ruby"
date: 2016-03-01 10:56:51 -0800
comments: true
categories: tools, sublime, ruby, optimization
Author: Umer Khan
---
## Rubocop

> *“A Ruby static code analyzer, based on the community Ruby style guide.”* **- Rubocop Github page**

[Rubocop](https://github.com/bbatsov/rubocop) is a code analyzer that enforces the rules presented in the [ruby style guide](https://github.com/bbatsov/ruby-style-guide) through static analysis. It helps you write code that adheres to the best practices in the Ruby community. You can install Rubocop by simply running ```$ gem install rubocop```. In order for rubocop to be executed by sublime and act as a linter, you will need to install the package "**SublimeLinter-rubocop**." To install the package do the following:
```
Open Sublime and press ctrl + shift + p (Windows, Linux) or cmd + shift + p (OS X)
Type "Package Control: Install Package" and select it
Wait for Sublime to load and then enter "rubocop"
Among the options, find "SublimeLinter-rubocop" and select it
```

If there are rules that you want to override then it's as simple as adding a .rubocop.yml file to the root of your project. In that file you can specify which default rubocop rules you would like to override. Here is an example of a rule that I'm overriding to prevent rubocop from linting lines with more than 80 characters:
```
Metrics/LineLength:
  Description: 'Limit lines to 80 characters.'
  StyleGuide: 'https://github.com/bbatsov/ruby-style-guide#80-character-limits'
  Enabled: false
```

## Packages
*All of thsese packages can be installed by following the steps listed above for Rubocop.*
### DashDoc (OS X users only)
[DashDoc](https://github.com/farcaller/DashDoc) adds Dash integration in to Sublime. For example if you want to look up the documentation for the ```before_action``` method then you just highlight it and press ```ctrl + h```. Doing this will open Dash and the ruby documentation for this method will be displayed.

### All Autocomplete
[All Autocomplete](https://github.com/alienhard/SublimeAllAutocomplete) expands the autocompletion feature in sublime to search all the files that are open in the editor rather than just the file that you are working on.

<img src='http://i.imgur.com/NdYkVTP.png' width="400px">

### GitGutter
[GitGutter](https://github.com/jisaacks/GitGutter) shows an icon in the gutter for any line that you have added, deleted, or modified.

![Git Gutter](http://i.imgur.com/SFn4GD0.png)

###Other Packages
Here are a few other packages that add many features and shortcuts to help you be a more efficient ruby developer: [Clipboard Manager](https://github.com/colinta/SublimeClipboardManager), [BeautifyRuby](https://github.com/CraigWilliams/BeautifyRuby), [ChangeQuotes](https://github.com/colinta/SublimeChangeQuotes), [CoffeeScript](https://github.com/Xavura/CoffeeScript-Sublime-Plugin), and [RSpec](https://github.com/SublimeText/RSpec).

##Settings
Sublime makes it easy to customize all aspects of the editor by specifying custom settings.


Here is a list of settings that I am using (see below). I would suggest using this as a starting point to tweek things to what works best for you:
```
{
  "auto_complete": true,
  "auto_complete_commit_on_tab": true,
  "caret_style": "solid",
  "copy_with_empty_selection": true,
  "draw_white_space": "all",
  "ensure_newline_at_eof_on_save": true,
  "font_size": 11,
  "highlight_line": true,
  "highlight_modified_tabs": true,
  "rulers":
  [
    80,
    120
  ],
  "tab_size": 2,
  "translate_tabs_to_spaces": true,
  "trim_trailing_white_space_on_save": true,
  "wide_caret": true,
  "word_separators": "./\\()\"'-:,.;<>~@#$%^&*|+=[]{}`~"
  "word_wrap": true,

}
```

####Sublime Text 3 Only:
This setting ensures that all files in your project are indexed. This means that you can use the *Goto Definition* and *Goto Symbol* features to go directly to the spot where something is defined. This is a huge time saver.
```
"index_files": true
```
