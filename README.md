# LinkUp

## Introduction

This is a project that allows the easy creation of files that link between applications on MacOS/iOS. A familiarity with Keyboard Maestro and Siri Shortcuts is assumed.

**Usage**: for example when in Mail, Maps, Calendar, Safari etc. easily create a todo or note with a useful title, content and link back to the original item (where supported).

On a Mac the process is:

1. **Capture**: Hit a [Keyboard Maestro](https://www.keyboardmaestro.com) hot key sequence in your source application (I use: CTRL + ⌥ + ⌘ + L).
2. **Choose**: a destination application from a menu.
3. **Create**: an entry in the destination app which opens populated with a title, note and link back to the original item if possible.

On iOS/iPadOS the process is:

1. **Capture**: share from the source application to a Siri Shortcut called ```ShareFrom-<AppName>```.
2. **Choose**: a destination application from a menu.
3. **Create**: an entry in the destination app which opens populated with a title, note and link back to the original item if possible.

The choose and Create steps are common between MacOS and iOS/iPadOS, using the same shortcuts.

For example, suppose you are in Safari on a Mac and you want to create a note from the current page. Just hit the hot key sequence, and choose Notes. 

![Step 2](step-2.png)

This creates the following Apple Notes note:

![Step 3](step-3.png)

The main difficulty is capture in circumstances where the source application may not conveniently provide the information we want. This will be discussed on a case by case basis later.

## Structure

This is a collection of Keyboard Maestro macros and Siri Shortcuts that fall into three main categories.

**1. Capture**

On MacOS capture is done with Keyboard Maestro (KM) macros (```Link-<AppName>```). KM gathers information from the application using a variety of dark spells, builds a dictionary with various parameters.

On iOS/iPadOS capture is done with via the system share menu to Siri Shortcuts (```ShareFrom-<AppName>```) configured as share targets.

Both of the above then call a common set of shortcuts (```LinkFrom-<AppName>```) that do whatever additional work is required to fill in any missing parameters as best they can to include:

- Title
- Note
- URL
- Latitude, Longitude (for locations)

These parameters are then passed to a single shortcut in step 2.

**2. Destination Choice**

This is a single common shortcut that prompts the user for the destination application, calling the final app specific creation shortcut in step 3.

**3. Create Chosen Item**

This is a set of shortcuts (```LinkTo-<AppName>```) that builds whatever content you asked for in step 2 and ideally opens it for verification.

## Supported Applications

Applications are either Sources (which can be linked from) or Destinations which can be linked to. 
### Safari (Source)

- KM: [Link-Safari](Link-Safari.kmmacros)
- Shortcut: [LinkFrom-WebPage](LinkFrom-WebPage.shortcut)

**URL**: In Safari it's quite easy to capture the URL, KM can easily capture the current URL and on iOS/iPadOS the url can be shared.

**Title**: In the shortcut we use URL and **Get Article using Safari Reader**. This provides the title.

**Note**: The article fetched above also contains the content, which can be passed to the local Apple AI Summarize feature. This as an option since it can go wrong if not enough content can be extracted from the URL.

### Mail (Source, Mac Only)

- KM: [Link-Mail](Link-Mail.kmmacros)
- Shortcut: [LinkFrom-Mail](LinkFrom-Mail.shortcut)

Unfortunately iOS/iPadOS do not provide a share menu from Mail or any particularly useful Shortcuts actions for accessing mail.

**URL**: KM has to use a fragment of AppleScript to get a mail URL.

**Title**: captured with KM.

**Note**: This is empty.

### Maps (Source)

- KM: [Link-Maps](Link-Maps.kmmacros)
- Shortcut: [LinkFrom-Maps](LinkFrom-Maps.shortcut)
- Shortcut: [ShareFrom-Maps](ShareFrom-Maps.shortcut)

Apple seems to have recently changed Maps to share a shortened URL rather than either a URL with parameters or a "Location" object for Shortcuts. Also the new Liquid Glass UI breaks the KM feature that can find the menu hidden behind the ```...``` by searching for an image - which is where Copy Coordinates hides on the Mac. This eliminates any documented way of extracting any information from what's shared from Maps. However after some experimentation I discovered that ```Expand URL``` in Shortcuts will take the shortened URL and return one that contains the name, address and coordinates.

**URL**: Use the shortened one.

**Title**: Use the **name** from the expanded URL.

**Latitude/Longitude**: From the expanded URL:

**Note**: The address from the expanded URL and also links to the same location in several other maps services like Google Maps, Open Street Map and the Ordnance Survey.

### Calendar (Source, Mac Only)

- KM: [Link-Calendar](Link-Calendar.kmmacros)
- Shortcut: [LinkFrom-Calendar](LinkFrom-Calendar.shortcut)

Unfortunately there is no useful share functionality in iOS/iPadOS.

On the Mac, selecting a calendar entry and activating Copy via KM will yield the title and the scheduling information on several lines in the clipboard. An annoying alert can pop up to warn if this is a repeating event and KM has to notice and dismiss this. The shortcut snips the title from the first line and then hunts for the calendar event by name, prompting for which one is wanted if it finds more than one. Once we have the event in the shortcut we can extract all the info we need for the note.

**Title**: From the Event found by looking up the title from KM.

**Note**: From the Event found by looking up the title from KM. Use any attached note text and the location.

**URL**: There is currently no URL Scheme for Calendar so we can't link back to the source. However if there is a URL associated with the event then that will be used.

### Obsidian (Source, Destination)

- [Link-Obsidian](Link-Obsidian.kmmacros)
- [LinkFrom-Obsidian](LinkFrom-Obsidian.shortcut)
- [ShareFrom-Obsidian](ShareFrom-Obsidian.shortcut)
- [LinkTo-Obsidian](LinkTo-Obsidian.shortcut)

When Obsidian is a source on the Mac, the KM macro assumes that the [Advanced URI Plugin](https://publish.obsidian.md/advanced-uri-doc/Home) is installed in Obsidian to ensure that the URL it extracts continues to work even if the if the source is renamed. This also links to the correct vault.

When Obsidian is used as a destination, the file is created using the default Obsidian URI scheme in a vault and folder defined in the shortcut, these will need to be modified to suit your needs and are defined at the top of the shortcut. 

Various front matter properties are added.

- id: a unique ID compatible with the Advanced URI plugin so that the new note can be opened on creation using it.
- location: If the source was Maps then a location property is created with Latitude and Longitude. This is compatible with the [Map View Plugin](https://github.com/esm7/obsidian-map-view) if used, but not required.

### OmniFocus (Source, Destination) 

When used as a source, ether on the Mac or iOS, OmniFocus provides a URL for the task. The shortcut can look up the task with that URL and extract the Title and Note.

Currently only tasks work as sources, I'd like to get tags, projects and folders working too.

When used as a destination, task creation is straightforward.

### Other Simple Destination Cases

These are pretty straight forward and don't need further explanation, making trivial use of Title, URL and Notes data captured from the Source.

- **Notes** (Destination).
- **[Tot](https://tot.rocks)** (Destination).
- **Webloc** (Destination) - A webloc file created on the desktop.
- **Clipboard**: (Destination).
- **Markdown** Link and Paragraph formats left in the clipboard.
