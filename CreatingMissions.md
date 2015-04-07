# Table of Contents

* [Introduction](#intro)
* [Text replacements](#replacement)
* [Basic mission characteristics](#basics)
* [Conditions](#conditions)
* [Source and destination filters](#filters)
* [Non-Player Characters (NPCs)](#npcs)
* [Triggers](#triggers)

<a name="intro"/>
# Introduction #

The basic syntax of a mission description is:

```html
mission <name>
    name <name>
    description <text>
    deadline (<days>)
    cargo (random | <name>) <number> (<number> (<probability>))
        illegal <fine>
    passengers <number> (<number> (<probability>))
    invisible
    (job | landing)
    repeat (<number>)
    clearance (<message>)
        ...
    infiltrating
    waypoint <system>
    to (offer | complete | fail)
        <condition> (<comp> <value>)
        (has | not) <condition>
        never
    (source | destination) <planet>
    (source | destination)
        planet <name>*
            <name>*
        system <name>*
            <name>*
        government <name>*
            <name>*
        attributes <name>*
            <name>*
        near <system> ((<min>) <max>)
        distance ((<min>) <max>)
    npc (save | kill | board | disable | "scan cargo" | "scan outfits" | evade | accompany)*
        government <name>
        personality <type>*
            <type>*
            confusion <amount>
        system (<system>)
        system
            system <name>*
                <name>*
            government <name>*
                <name>*
            near <system> ((<min>) <max>)
            distance (<min>) <max>
        dialog <text>
            <text>*
        conversation <name>
        conversation
            ...
        ship <model> <name>
        ship <model>
            ...
        fleet <name>
        fleet
            ...
    on (offer | complete | accept | decline | fail | visit | enter <system>)
        dialog <text>
            <text>*
        conversation <name>
        conversation
            ...
        outfit <outfit> (<number>)
        payment (<value>)
        <condition> (= | += | -= | ++ | --) (value)
        (set | clear) <condition>
        event <name> (delay)
```

Each of these parts of the mission description is described in detail below.

<a name="replacement"/>
# Text replacements #

Certain characteristics of a mission, such as the cargo or the destination planet, may be chosen at random. In order to refer to those randomly chosen elements  in descriptive text, you can use the following placeholders:

* `<commodity>` = name of commodity being carried
* `<tons>` = "1 ton" or "N tons"
* `<cargo>` = "`<tons>` of `<commodity>`"
* `<bunks>` = the number of passengers
* `<passengers>` = "passenger" or "passengers"
* `<fare>` = "a passenger" or "N passengers"
* `<origin>` = planet where mission was offered
* `<planet>` = destination planet
* `<system>` = destination system
* `<destination>` = "`<planet>`, in the `<system>` system"
* `<payment>` = "1 credit" or "N credits"
* `<date>` = the deadline for the mission (in the format "Day, DD Mon YYYY")
* `<day>` = the deadline in conversational form ("the DDth of Month")
* `<npc>` = the name of the first NPC in the mission description
* `<first>` = your first name
* `<last>` = your last name
* `<ship>` = the name of your flagship

These placeholders will be substituted in any text in the following places:

* the mission name
* the mission description
* dialog messages contained in the mission
* conversations contained in the mission

For example, the mission description might be, "Deliver `<cargo>` to `<destination>`, by `<date>`."

<a name="basics"/>
# Basic mission characteristics #

```html
mission <name>
```

The mission name must be unique.

```html
name <name>
```

Because the mission name must be unique, if you want to have the same name displayed for two different missions, you can specify the display name separately. This is useful, for example, if you want to have multiple jobs that all get displayed as "ferry passengers to `<planet>`".

```html
description <text>
```

This is a short description of the mission, with enough detail to make it clear to the player what they need to do to complete the mission, and what could cause the mission to fail.

```html
deadline (<days>)
```

The number of days you have to complete the mission. If the number of days is left out (i.e. it is just the word "deadline"), that means use the default deadline (2 days \* number of hyperspace jumps to the destination). In a saved game, the `<days>` amount is stored as an absolute date instead of a relative number, e.g. "deadline 29 1 3014".

```html
cargo (random | <name>) <number> (<number> (<probability>))
    illegal <fine>
```

This specifies the cargo that you are carrying. If `<name>` is one of the standard commodity names defined in the "trade" data, it will be replaced by a random one of the specific names for that commodity, e.g. "Food" might be replaced by "canned fruit" or "evaporated milk".

If the given cargo name is the word "random", a random basic commodity type will be chosen based on the relative commodity prices at the source and destination system. That is, it will attempt to pick a commodity that would make sense as an export from the one system to the other.

If two amounts are given instead of one, that means that a random amount should be chosen in between those two numbers (inclusive).

If three numbers are given, a random number will be chosen by adding the first number to a random number chosen from a negative binomial distribution with the given number of successes needed and probability. This produces numbers that are generally somewhat low but can occasionally be quite high, if you want to every once in a while have massive cargo missions, for example.

If the cargo is marked as "illegal," governments that care about such things will levy the given fine against you if you are caught carrying this cargo. If the fine is negative, being caught with this cargo while in flight is counted as an "atrocity" (the government that catches you immediately becomes your enemy, no matter how good your reputation was previously), and if you are caught in a spaceport with the cargo the result is a death sentence (game over).

```html
passengers <number> (<number> (<probability>))
```

This specifies the number of passengers. As with the cargo specification, if there are two or three numbers they are used to pick a random number.

```html
invisible
```

This specifies that the mission does not show up in the player's list of missions, and cannot be canceled.

```html
(job | landing)
```

This specifies where this mission will be shown, if someplace other than the spaceport. If it is a "job", it will appear only if included in the job board (which only happens if the current planet names this mission as one of its job templates).

If this mission is to be shown at "landing," it shows up as soon as you land instead of waiting for you to visit the spaceport. This can be used, for example, to show a special conversation the first time you land on a particular planet or on any planet belonging to a certain species. It can also be used for a continuation of an active mission.

```html
repeat (<number>)
```

If the word "repeat" appears by itself, this mission can be offered any number of times. If a number is given, that is the maximum number of times this mission can be offered. By default, each mission can only be offered once, so having a "repeat 1" specified is unnecessary.

If you want a mission to be offered any number of times but to limit the number of instances of the mission that can be active concurrently, you can decrement the "`<mission name>`: offered" condition (described below) whenever the mission is completed, failed, or declined.

```html
clearance (<message>)
    ....
```

This gives you landing clearance on the destination planet, even if normally you would not be allowed to land there, or would have to pay a bribe.

If a "clearance" tag appears followed by a message, if you hail your destination planet that message will be shown as the initial text, landing permission will be granted, and you will not have to pay a bribe. The message will be shown even for friendly planets so that this can be used to customize their hail, e.g. "Welcome back, Captain `<last>`! Everyone's waiting for you at the spaceport."

If a "clearance" tag appears with no message, clearance is automatically granted without needing to hail the planet. You can use this, for example, if the mission involves secretly landing on a planet under the cover of darkness and the last thing you want to do is inform the authorities that you're coming.

If a specific destination is given (i.e. "destination `<planet>`") and no clearance is included in the mission, you must pay a bribe if you are not allowed to land on that planet. (Sometimes bribing the authorities could be part of the mission plot.)

If the destination is specified via a filter, the filter will not match planets you cannot land on unless this mission contains a "clearance" tag. This may make it impossible for a particular mission to be offered.

The "clearance" tag may have child entries that specify a location filter, the same as the "source" and "destination" tags described below. In this case, you have clearance on all planets that match that filter, in addition to on the destination planet.

```html
infiltrating
```

This indicates that you do not have access to any of the services on the destination planet, including refueling or repairing; all you can do is complete your mission and then leave. This is useful for missions that involve you landing secretly somewhere other than the spaceport; it wouldn't make sense to then allow the player to then walk around the spaceport or purchase things.

```html
waypoint <system>
```

This specifies a system which you must fly through in order to complete the mission. You do not have to land on any planets or spend any amount of time there. Waypoints are marked on the map in red until they have been visited; then they disappear.

<a name="conditions"/>
# Conditions #

"Conditions" are named values that represent things the player has done. Conditions start out with a value of zero, and can only have integer values. Conditions can have almost any name you want, as long as you make sure not to use the same name in two places. A few names are reserved for special purposes:

* `"<mission name>: offered"`, where `<mission name>` is replaced with the name of any mission. This is incremented whenever a mission is offered to you, and is used by the "repeat" check to make sure a mission is not offered too many times.
* `"<mission name>: active"`, where `<mission name>` is replaced with the name of any mission. This is incremented when you accept a mission, and decremented when you complete, or fail it.
* `"<mission name>: done"` is set when a mission is successfully completed.
* `"reputation: <government>"` is set to your current reputation with the given government, rounded down to a whole number.
* "reputation" is your reputation in the current system, rounded down.
* "ships: `<category>`" is the number of ships you have of each category (Transport, Light Freighter, Heavy Freighter, Interceptor, Light Warship, Heavy Warship, Fighter, Drone).
* "combat rating" is your current combat rating (sum of the base crews of all the ships you have disabled).
* "random" is a random number between 0 and 99. This can be used to make a mission only sometimes appear when all other conditions are met.

Conditions are checked at two times when processing a mission: when determining whether the mission can be offered right now (in the "to offer" tag), and when determining whether it has been completed successfully (in the "to complete" tag):

```html
to (offer | complete | fail)
    <condition> (<comp> <value>)
    (has | not) <condition>
    never
```

The `<comp>` comparison operator can be `==`, `!=`, `<`, `>`, `<=`, or `>=`. If only the condition name is given, the comparison is assumed to be "!= 0". As a special shortcut, you can write "has `<condition>`" instead of "`<condition>` != 0", or "not `<condition>`" instead of "`<condition>` == 0". The "never" condition always evaluates to false, so it can be used to create a mission that can never succeed.

Conditions can be changed when you are offered a mission or when you accept, decline, fail, or complete it, as described later in this document. You can "chain" missions together by having one depend on the "`<mission name>`: done" condition set by a previous mission. Since all your existing missions complete before new ones are offered, that condition will be set before the check is done for what new missions are available.

A mission will not be offered if any of the "to fail" conditions are met, and will fail if it is active and one of those conditions changes so that it is satisfied.

<a name="filters"/>
# Source and destination filters #

Each mission starts on a planet, and ends on a planet, usually different from where it started. You can either specify one particular planet, or give a set of constraints that the planet must match:

```html
(source | destination) <planet>
(source | destination)
    planet <name>*
        <name>*
    government <name>*
        <name>*
    attributes <name>*
        <name>*
    near <system> ((<min>) <max>)
    distance (<min>) <max>
```

If no destination is specified, the destination is the same as the source planet. This can be useful for missions that should end on the same planet they start on, such as "Kill pirate ship X and return here for payment." If no source is specified, the mission will be offered whenever its "to offer" conditions are satisfied; this can be used to create a mission that is offered as soon as you complete another.

Each entry in the source or destination specification acts as a filter:

```html
planet <name>*
    <name>*
```

This says that the planet must be one of the named planets. The list of names can either be all on one line, or split between multiple lines if it is particularly long; the subsequent lines must be indented so that they are "children" of the "planet" node. As with most of these filters, you can also have more than one "planet" entry, in which case the planet chosen must be in any one of the lists.

```html
system <name>*
    <name>*
```

The system must be one of the items in this list. You can use this if you do not want to bother to look up what planets are in the system, but the intended use is for the NPC location filter as described later.

```html
government <name>*
    <name>*
```

The planet must be in a system owned by the given government. Again, the list can be all on one line, or multiple indented lines.

```html
attributes <name>*
    <name>*
```

The planet must have one of the given attributes (e.g. "dirt belt", "urban", "rich", "tourism", etc.).

Unlike the other filters, if multiple "attribute" tags appear, the planet must contain at least one attribute from each of the lists. For example, this means the planet must be urban or rich:

```html
attributes urban rich
```

but this means it must be urban _and_ rich:

```html
attributes urban
attributes rich
```

There are also ways of specifying how far the system is from a particular location, or from the current location:

```html
near <system> ((<min>) <max>)
```

If one number is given, the planet must be within that number of jumps from the given system (which includes the given system itself). If two numbers are given, the distance from the given system must be at least as high as the first number, and no more than the second. If no numbers are given, the planet must be in the given system or one of the systems it is linked to; this is equivalent to giving distances of 0 and 1.

```html
distance (<min>) <max>
```

This is the same as the "near" tag, but gives distances relative to the origin planet. (So, this tag only makes sense within a "destination" filter, not within a "source" filter or a "clearance" filter.)

<a name="npcs"/>
# Non-Player Characters (NPCs) #

NPCs are ships that are associated with the mission in some way. This includes friendly ships the player must protect, and hostile ships the player must fight off or destroy:

```html
npc (save | kill | board | disable | "scan cargo" | "scan outfits" | evade | accompany)*
    government <name>
    personality <type>*
        <type>*
        confusion <amount>
    system <system>
    system
        system <name>*
            <name>*
        government <name>*
            <name>*
        near <system> ((<min>) <max>)
        distance ((<min>) <max>)
    dialog <text>
        <text>*
    conversation <name>
    conversation
        ...
    ship <model> <name>
    ship <model>
        ...
    fleet <name>
    fleet
        ...
```

Each "npc" tag may have one or more tags following it, specifying what the player must do with the given NPC:

* save: The mission fails if the given NPC is destroyed.
* kill: To complete the mission, the given NPC must be dead.
* board: To complete the mission, the player must board the given NPC. If the ship is destroyed before being boarded, the mission fails.
* disable: To complete the mission, the player must disable the given NPC.
* "scan cargo": To complete the mission, the player must scan the given NPC's cargo. If the NPC is destroyed before being scanned, the mission fails.
* "scan outfits": Same, but the player must use an outfit scanner instead of a cargo scanner.
* evade: you cannot complete the mission if any members of this NPC are in the same system as you.
* accompany: you can only complete the mission if all members of this NPC are in the same system as you.

```html
government <name>
```

This specifies what government all the ships connected to this NPC specification will have. If no government is given, they are set to the player's government, as escorts.

```html
personality <type>*
    <type>*
    confusion <amount>
```

This defines the NPC "personality", using the same personality type flags as are used in fleet specifiers. The "confusion" tag is a special value, giving the inaccuracy in pixels of the ship's targeting systems; the default value is 10 pixels.

The valid personality types (which can be combined) are:

* pacifist: will not attack under any circumstances.
* forbearing: will not attack unless its shields are reduced to below 90%.
* timid: does not join other people's fights; only attacks targets that are nearby and targeting it.
* disables: tries to disable enemy ships rather than destroying them.
* plunders: will board and pillage enemy ships before destroying them.
* heroic: more likely to join fights when an ally is threatened.
* staying: never leaves the system it starts out in.
* entering: enters its starting system via hyperspace instead of appearing there.
* nemesis: only attacks the player's ships.
* surveillance: scans random ships and visits random planets in system.
* uninterested: does not follow the player's flagship around (i.e. does not behave like an escort).
* waiting: starts out in space in the given system, then follows you.
* derelict: starts out disabled.
* fleeing: tries to run away from the player.

In addition, if an NPC is specified as starting out in your current system and its personality is _not_ "staying" or "waiting", it will take off from the planet along with you (e.g. a ship you are escorting). A ship that is "entering" the current system might, for example, be a pirate raid chasing the fleet you are escorting, and a ship "staying" in a certain system might be a target you must locate for a bounty hunting mission. (Any ship that is not "staying" will actively seek the player out if it is in a different system.)

```html
system (<system>)
    system <name>*
        <name>*
    government <name>*
        <name>*
    near <system> ((<min>) <max>)
    distance ((<min>) <max>)
```

This specifies what system the NPC starts out in. If no system is specified either by name or through the "near" or "distance" child tags, the system will be the current system.

The "system", "government", "near", and "distance" filters operate the same way they do in the descriptions in the previous section, and can be used instead of naming a particular system. For example, you could have the NPC start out in any Pirate system, or within two jumps of the current system.

```html
dialog <text>
    <text>*
conversation <name>
conversation
    ...
```

This defines a dialog or conversation to be shown when you have first satisfied all the requirements of a given NPC. For more details on the syntax, see the "Triggers" section below.

If you want to retrieve passengers or cargo by boarding a ship, set up the mission so that you are considered to be carrying them from the very start (for example, the cargo might be called "reserved mission space" or "mission cargo"). Otherwise, it would be possible for the player to board a ship and then discover they do not have enough cargo or passenger space to complete the mission.

```html
ship <model> <name>
ship <model>
    ...
```

This specifies a single ship as an NPC. There are two possible formats. The first format says to create a generic ship of the given model type (or named variant), such as "Falcon", or "Star Barge (Armed)". In that case, the ship will have the given name.

The second option is to give an entire ship specification, which can optionally include things like number of crew members remaining if you want to create a ship that starts out as a disabled "ghost ship."

```html
fleet <name>
fleet
    ...
```

This specifies an entire fleet of ships. The first format refers to one or the standard fleets, such as "pirate raid" or "Small Republic". The second format gives a custom fleet, using the same syntax as  normal "fleet" data entry. Every ship in the fleet will have the requirements given in the first line (such as "kill" or "save").

<a name="triggers"/>
# Triggers #

A mission can also specify what happens at various key parts of the mission:

```html
on (offer | complete | accept | decline | fail | visit | enter <system>)
    dialog <text>
        <text>*
    conversation <name>
    conversation
        ...
    outfit <outfit> (<number>)
    payment (<value>)
    <condition> (= | += | -= | ++ | --) (value)
    (set | clear) <condition>
    event <name> (delay)
```

There are seven events that can trigger a response of some sort:

* offer: when the initial mission is offered. This is the place to put the conversation or dialog that introduces the mission.
* complete: when the mission is completed. This is when the player gets paid.
* accept: if the player agrees to accept a mission.
* decline: if the player decides to decline a mission.
* fail: if the mission fails.
* visit: you land on the mission's destination, and it has not failed, but you have also not yet done whatever is needed for it to succeed.
* enter `<system>`: your ship enters the given system for the first time since this mission was accepted.

Some of the events below usually only make sense for certain triggers. In particular, dialogs and conversations can be shown when a mission is offered, but not in response to it being accepted or declined; just add the appropriate text to the offer conversation instead.

```html
dialog <text>
    <text>*
```

This gives a message to be displayed in a dialog message to the user. If the trigger is "on offer", the dialog will have "accept" and "decline" buttons. Otherwise, it is a purely informational message and only an "okay" button is shown.

Each token following the "dialog" tag will be a separate paragraph. The first token may appear either on the same line or indented on a subsequent line.

As mentioned previously, text replacement is done on keywords like "`<destination>`" and "`<payment>`" within the dialog text.

```html
conversation <name>
conversation
    ...
```

This specifies that a conversation will be shown to the player at this point in the mission. When a mission is being offered, the conversation can return "accept" or "decline"; conversations can also return special values like "die" (if the conversation ends with the player dying) or "launch" (if the player should take off from the planet immediately).

As with the dialogs, text substitution is done throughout the conversation.

The syntax for conversations is described [here](WritingConversations).

```html
outfit <outfit> (<number>)
```

At this point in the mission, the named ship outfit (or some number of them, if a number is given) is installed in the player's flagship, or placed in the player's cargo if the flagship has no outfit space for it. If the number is negative, outfits are taken away. If a mission removes outfits in its "complete" phase, it cannot be completed unless that outfit exists. This makes it possible to loan the player an outfit for the duration of a mission, or to require that the player disable a non-NPC ship and steal a particular piece of technology from it.

If the outfit cannot be installed due to lack of space, a warning message will be shown so the player knows that the outfit is not actually active (and may in fact be lost if they leave the planet).

```html
payment (<value>)
```

If the "payment" tag appears on its own, the player is paid the default amount for this mission, which is equal to:

(# of jumps from origin to destination + 1) \* (1500 \* (# of passengers) + 150 \* (tons of cargo))

If a value is given, that is the player's payment for reaching this step of the mission. Normally, a "payment" value would only be given in the "on complete" section, but you can also have a negative value to be subtracted if the player fails the mission, or you could use a "payment" to advance the player some money when they first accept a mission. If the "on complete" payment is negative, the player cannot complete the mission if they have fewer than that number of credits.

```html
<condition> (= | += | -= | ++ | --) (value)
(set | clear) <condition>
```

This is used to adjust the "conditions" described previously, which form the requirements for other missions. You can either set the condition to a certain value (using the "=" operator), or add or subtract a value using "+=" or "-=". As a shortcut, "++" means "+= 1" and "--" means "-= 1". You must place a space between the condition, the operator, and the value in order for the parser to interpret it correctly.

The "set" and "clear" tags are shortcuts for "= 1" and "= 0".

To change the player's reputation or combat rating, you may alter the "reputation", "`reputation: <government>`", or "combat rating" conditions. For example, a bounty hunting mission might automatically alter your reputation on whatever planet the mission ends on ("reputation"), and a mission for the Navy might improve your reputation with the Republic but reduce your reputation with the Free Worlds.

```html
event <name> (<delay>)
```

This specifies that the given event happens at this point in the mission. Events may permanently alter planets or solar systems. If a delay is given, the event will occur that number of days from now, instead of happening immediately.