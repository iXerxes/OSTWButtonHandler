# OSTW Button Handler Module - Version 2!

## A relatively light-weight [OSTW](https://github.com/ItsDeltin/Overwatch-Script-To-Workshop) module used to precisely manage button presses from a player (or bot).

### What's the point?
If you have a mode that relies a lot on various key-press-activated commands, this might make it easier to manage. Such as "one click for X" or "two clicks for Y".

### What can it do:
- Track multiple, chained clicks from a player - If your script requires a user to press a button a certain number of times to trigger an action.
- Track how long a button has been held for - Allows you to accurately track how long a button has currently been held for.
- Precision - Tracking the hold length is accurate up to about ~10-20ms. Ping doesn't seem to have much affect on this, either.

### Multi-Clicks
Multi-Clicks track when a button is pressed multiple times in quick succession. You can use this to trigger different events with the same button. Similar to how a single and double click works. The default is set to 300ms; presses within ~300ms of each other are tracked as a multi-click.

___

## How To Use

#### Make your rule and then use the following macros as your conditions:

- [<b>Press Count</b>](#button-press-count--button-hold-length) - Get the number of times a button has been pressed (multi-clicks); __returns Number__.<br>
`Number ButtonPressCount(Player player, EButtonInput button)`
  - `player`: The player you want to check.
  - `button`: The button you want to check. Follows the same structure as the OSTW `Button` enum.

  Example: `If (ButtonPressCount(player, button) >= 3)`

<br>

- [<b>Hold Length</b>](#button-press-count--button-hold-length) - Get the length of time a button has been held for (in seconds); __returns Number__.<br>
`Number ButtonHoldLength(Player player, EButtonInput button)`
  - `player`: The player you want to check.
  - `button`: The button you want to check. Follows the same structure as the OSTW `Button` enum.

  Example: `If (ButtonHoldLength(player, button) >= 1.5)`

<br>

- [<b>Press & Hold</b>](#button-press--hold) - Combines `Press Count` & `Hold Length` into one conidtion/macro; __returns Boolean__.<br>
`Boolean ButtonPressAndHold(Player player, EButtonInput button, Number clickCount, Number holdLength)`
  - `player`: The player you want to check.
  - `button`: The button you want to check. Follows the same structure as the OSTW `Button` enum.
  - `clickCount`: How many times the button should be pressed.
  - `holdLength`: How long the button should be held for.

  Example: `If (ButtonPressAndHold(player, button, 3, 1.5))`



### Special Conditions
In this new version, there have been some changes made that allow for better control and accuracy. With that, comes two more (slightly confusing) conditions/macros.

- [<b>Post-Hold Length</b>](#button-post-hold-length) - Get the length of time a button was held for ***after*** being released; __returns Number__.<br>
`Number ButtonPostHoldLength(Player player, EButtonInput button)`
  - `player`: The player you want to check.
  - `button`: The button you want to check. Follows the same structure as the OSTW `Button` enum.

  Example: `If (ButtonPostHoldLength(player, button) >= 1.5)`

  > - Unlike `Hold Length`, this is the final, absolute duration (in seconds) that a button was held for, *after* it was released.
  > - Only available for 1 tick after the button has been released.

  <br>

- [<b>Press & Post-Hold</b>](#button-press--post-hold) - Combines `Press Count` & `Post-Hold Length` into one condition/macro; __returns Boolean__.<br>
`Boolean ButtonPressAndPostHold(Player player, EButtonInput button, Number clickCount, Number holdLength)`
  - `player`: The player you want to check.
  - `button`: The button you want to check. Follows the same structure as the OSTW `Button` enum.
  - `clickCount`: How many times the button should be pressed.
  - `holdLength`: How long the button should be held for.

  Example: `If (ButtonPressAndPostHold(player, button, 3, 1.5))`

  > - Same as `Press & Hold`, but it uses the `Post-Hold Length` instead.<br>
  > - The normal `Press & Hold` will trigger the condition when ever the button's hold-duration reaches a certain threshhold. The difference is: this will only tigger the condition **once the button has been released**.

  ___

## Examples

### Button Press Count & Button Hold Length

OSTW:

```hs
rule: "Double-Click Interact"
Event.OngoingPlayer
if (ButtonPressCount(EventPlayer(), EButtonInput.Interact) == 2)
{
    SmallMessage(EventPlayer(), "You double-clicked Interact!");
}

rule: "Triple-Click Melee & hold for 2.5 seconds"
Event.OngoingPlayer
if (ButtonPressCount(EventPlayer(), EButtonInput.Melee) == 3)
if (ButtonHoldLength(EventPlayer(), EButtonInput.Melee) >= 2.5)
{
    SmallMessage(EventPlayer(), "You triple-clicked Melee and held it for 2.5 seconds!");
}
```

Workshop Output:

```hs
rule("Double-Click Interact")
{
    event {Ongoing - Each Player;All;All;}

    conditions
    {
        X Component Of(Value In Array(Player Variable(Event Player, _ButtonHandler), 5)) == 2;
    }
    actions
    {
        Small Message(Event Player, Custom String("You double-clicked Interact!"));
    }
}

rule("Triple-Click Melee & hold for 2.5 seconds")
{

    event {Ongoing - Each Player;All;All;}

    conditions
    {
        X Component Of(Value In Array(Player Variable(Event Player, _ButtonHandler), 8)) == 3;
        If-Then-Else(Value In Array(Player Variable(Event Player, _ButtonHandler), 8), Subtract(Global Variable(_BH_Clock), Y Component Of(Value In Array(Player Variable(Event Player, _ButtonHandler), 8))), 0) >= 2.5;
    }
    actions
    {
        Small Message(Event Player, Custom String("You triple-clicked Melee and held it for 2.5 seconds!"));
    }
}
```

### Button Press & Hold

OSTW:

```hs
rule("Double-Click Primary Fire and hold for 5 seconds")
{

    event {Ongoing - Each Player;All;All;}

    conditions
    {
        And(Compare(X Component Of(First Of(Player Variable(Event Player, _ButtonHandler))), >=, 2), Compare(If-Then-Else(First Of(Player Variable(Event Player, _ButtonHandler)), Subtract(Global Variable(_BH_Clock), Y Component Of(First Of(Player Variable(Event Player, _ButtonHandler)))), 0), >=, 5)) == True;
    }
    actions
    {
        Small Message(Event Player, Custom String("You double-clicked Primary Fire and held it for 5 seconds!"));
    }
}
```

Workshop Output:

```hs
rule("Double-Click Primary Fire and hold for 5 seconds")
{
    event{Ongoing - Each Player;All;All;}

    conditions
    {
        And(X Component Of(First Of(Player Variable(Event Player, buttonHandler))), Compare(Subtract(Global Variable(BH_gameClock), Y Component Of(First Of(Player Variable(Event Player, buttonHandler)))), >=, 5)) == True;
    }
    actions
    {
        Small Message(Event Player, Custom String("You double-clicked Primary Fire and held it for 5 seconds!"));
    }
}
```

### Button Post-Hold Length

OSTW:

```hs
rule: "Hold Crouch for at least 1.5 seconds and then release."
Event.OngoingPlayer
if (ButtonPostHoldLength(EventPlayer(), EButtonInput.Crouch) >= 1.5)
{
    SmallMessage(EventPlayer(), "You held crouch for at least 1.5 seconds, and then released!");
}
```

Workshop Output:

```hs
rule("Hold Crouch for at least 1.5 seconds and then release.")
{

    event{Ongoing - Each Player;All;All;}

    conditions
    {
        Z Component Of(Value In Array(Player Variable(Event Player, _ButtonHandler), 7)) >= 1.5;
    }
    actions
    {
        Small Message(Event Player, Custom String("You held crouch for at least 1.5 seconds, and then released!"));
    }
}
```

### Button Press & Post-Hold

OSTW:

```hs
rule: "Double-Click Reload, and then hold for hold for at least 3 second before releasing."
Event.OngoingPlayer
if (ButtonPressAndPostHold(EventPlayer(), EButtonInput.Reload, 2, 3))
{
    SmallMessage(EventPlayer(), "You double-clicked Reload, held it for at least 3 seconds, and then released!");
}
```

Workshop Output:

```hs
rule("Double-Click Reload, and then hold for hold for at least 3 second before releasing.")
{

    event{Ongoing - Each Player;All;All;}

    conditions
    {
        And(Compare(X Component Of(Value In Array(Player Variable(Event Player, _ButtonHandler), 9)), >=, 2), Compare(Z Component Of(Value In Array(Player Variable(Event Player, _ButtonHandler), 9)), >=, 3)) == True;
    }
    actions
    {
        Small Message(Event Player, Custom String("You double-clicked Reload, held it for at least 3 seconds, and then released!"));
    }
}
```

  ___

### Pros:

1. It's very accurate within about ~20ms. You can use an external macro program to accurately test timings.
2. Previously, hold lengths below the multi-click timeout were unreliable, and somewhat broken. You can now check the post-hold length to get the exact length of time a button was held for, before it was released.
3. Very simple to use in an OSTW project.
4. It does stuff.

### Cons:

1. Trying to trigger an action based on a multi-click will also trigger those bound to a lesser click-requirement; unless your hold length requirement is greater than the multi-click timeout.

    > **Example:**
    >
    > - I want to add 1 to a number for a single-click;
    > - And add 2 to a number for a double-click.
    >
    > When double-clicking, both the single-click action and the double-click action will be triggered.

    You can get around this by using the new `Post-Hold Length` condition, and checking that the button was held for more than a certain length of time.

2. *Slightly minor, but expected* - Trying to rapidly trigger an action with a button press wont work due to the multi-click timeout.
   > For example: If you want to quickly trigger a double-click action multiple times, you have to wait the duration of multi-click timeout before you can double-click again.
   >
   > You can lessen this by reducing the multi-click timeout.

3. I made it.